# AgentLoop

> Understand-Anything 知识图谱: 26 节点, 25 边, 7 层
> 节点 ID：`agent_loop` | 类型：class | 复杂度：complex
> 知识图谱摘要：核心对话 Agent 循环：管理 LLM 调用、工具执行、上下文折叠、流式响应

## 1. 模块定位
核心对话 Agent 循环：管理 LLM 调用、工具执行、上下文折叠、流式响应

## 2. 源码文件清单
| 文件 | 行数 | 类 | 顶层函数 |
|------|------|-----|---------|
| `deeptutor/agents/chat/agent_loop.py` | 667 | InlineThinkFilter, AgentLoopState, LLMCallResult, LoopOutcome, AgentLoop | `_last_context_checkpoint_summary` |

## 3. 类与函数颗粒度解析
### 文件：`deeptutor/agents/chat/agent_loop.py` (667 行)
#### 类定义
- **InlineThinkFilter** (第 64 行): Incremental ``<think>``/``<thinking>`` splitter for streamed content.
  - 方法：
    - `__init__()` (第 75 行): 
    - `feed()` (第 79 行): Consume *chunk*; return ``(kind, text)`` segments, kind in
    - `flush()` (第 107 行): Release whatever is still buffered (stream ended).
    - `_kind()` (第 115 行): 
- **AgentLoopState** (第 120 行): Turn-level counters shared across the loop's rounds.
- **LLMCallResult** (第 129 行): 
- **LoopOutcome** (第 136 行): Result of running the turn's loop.
- **AgentLoop** (第 149 行): Run one chat turn as a single agent loop over one conversation.
  - 方法：
    - `__init__()` (第 152 行): 
    - `run()` (第 169 行): 
    - `_clean()` (第 221 行): 
    - `_run_loop()` (第 226 行): Run rounds of one LLM call + tool dispatch over *messages*.
    - `_fold_context_checkpoint()` (第 351 行): 
    - `_forced_finish()` (第 371 行): 
    - `_finalize_finish()` (第 413 行): 
    - `_call_llm()` (第 430 行): 
    - `_create_response_stream()` (第 593 行): 
#### 顶层函数
- `_last_context_checkpoint_summary()` (第 646 行): 

## 4. 与其他实体的关系
- `agent_pipeline` → `contains` → `agent_loop`: Pipeline 管理 AgentLoop
- `agent_loop` → `calls` → `svc_llm`: AgentLoop 调用 LLM
- `agent_loop` → `calls` → `tools_builtin`: AgentLoop 调用工具
- `agent_loop` → `depends_on` → `core_agentic`: AgentLoop 依赖 Agentic 框架
- `cap_solve` → `calls` → `agent_loop`: 解题能力通过 Agent 执行

## 5. 关键设计理解
（此处预留，后续根据源码进一步补充具体逻辑流程。）

## 6. 源码逐行解读

### 6.1 核心数据结构
- `LoopState`：记录循环状态（narrating / finishing / paused / terminated）
- `Round`：每一轮 LLM 调用，包含 content 块和 tool calls
- `DispatchOutcome`：工具分发结果，决定下一轮是否继续

### 6.2 主循环流程（agent_loop.py）
```text
while budget > 0:
    response = await llm_client.complete(messages)
    if response.tool_calls:
        results = await dispatch_tool_calls(response.tool_calls)
        messages.extend(results)
    else:
        finish_round(response)
        break
```

### 6.3 关键设计点
- **单循环设计**：一次对话 turn 内只有一个 while 循环，避免多轮 respond/explore 分离。
- **工具调用即叙事**：任何一轮的工具调用文本都是 `narration`，不是最终答案。
- **预算控制**：通过 `max_rounds` 防止无限循环，超限时强制 finish。
- **上下文折叠**：当 messages 超过 context window 时，触发 UnifiedContext.fold。

## 7. 常见问题
- 为什么一轮内可能调用多次 LLM？因为每次工具调用结果都要返回给 LLM 再决策。
- 流式响应如何保证前端能实时看到内容？通过 StreamBus 把 LLM 流分发给 SSE。

## 8. AgentLoop 关键方法源码

### 类 `InlineThinkFilter` (第 64 行)
Incremental ``<think>``/``<thinking>`` splitter for streamed content.

#### `__init__()` (第 75 行)


```python
        self._buffer = ""
        self._in_think = False
```

#### `feed()` (第 79 行)
Consume *chunk*; return ``(kind, text)`` segments, kind in

```python
        """Consume *chunk*; return ``(kind, text)`` segments, kind in
        ``{"content", "thinking"}``. May hold back a partial trailing tag
        until the next chunk (``flush`` releases it at stream end)."""
        self._buffer += chunk
        segments: list[tuple[str, str]] = []
        while True:
            pattern = _THINK_CLOSE_RE if self._in_think else _THINK_OPEN_RE
            match = pattern.search(self._buffer)
            if match is None:
                break
            if match.start() > 0:
                segments.append((self._kind(), self._buffer[: match.start()]))
            self._buffer = self._buffer[match.end() :]
            self._in_think = not self._in_think
        emit_upto = len(self._buffer)
        tag_start = self._buffer.rfind("<")
        if (
            tag_start != -1
            and len(self._buffer) - tag_start <= _TAG_HOLDBACK_CHARS
            and ">" not in self._buffer[tag_start:]
        ):
            emit_upto = tag_start
        if emit_upto > 0:
            segments.append((self._kind(), self._buffer[:emit_upto]))
            self._buffer = self._buffer[emit_upto:]
        return segments
```

#### `flush()` (第 107 行)
Release whatever is still buffered (stream ended).

```python
        """Release whatever is still buffered (stream ended)."""
        if not self._buffer:
            return []
        segments = [(self._kind(), self._buffer)]
        self._buffer = ""
        return segments
```

#### `_kind()` (第 115 行)


```python
        return "thinking" if self._in_think else "content"
```

### 类 `AgentLoopState` (第 120 行)
Turn-level counters shared across the loop's rounds.

### 类 `LLMCallResult` (第 129 行)


### 类 `LoopOutcome` (第 136 行)
Result of running the turn's loop.

### 类 `AgentLoop` (第 149 行)
Run one chat turn as a single agent loop over one conversation.

#### `__init__()` (第 152 行)


```python
        self,
        *,
        pipeline: "AgenticChatPipeline",
        context: UnifiedContext,
        stream: StreamBus,
        client: Any,
        enabled_tools: list[str],
        tool_schemas: list[dict[str, Any]] | None,
    ) -> None:
        self.pipeline = pipeline
        self.context = context
        self.stream = stream
        self.client = client
        self.enabled_tools = enabled_tools
        self.tool_schemas = tool_schemas
```

#### `run()` (第 169 行)


```python
        state = AgentLoopState()
        # Optional async pre-pass briefings (e.g. explore_context) run BEFORE
        # the answer stage so they form their own preceding activity group and
        # their grounding can ride in the loop's user-message seed.
        capability_briefing = await self.pipeline._capability_pre_loop_briefings(
            self.context, self.stream
        )
        async with self.stream.stage(LOOP_STAGE, source="chat"):
            seed_block = await self.pipeline._retrieve_kb_seed_block(self.context, self.stream)
            capability_seed = self.pipeline._capability_pre_loop_seed(self.context)
            seed_block = "\n\n".join(
                block
                for block in (
                    seed_block.strip(),
                    capability_seed.strip(),
                    capability_briefing.strip(),
                )
                if block
            )
            messages = self.pipeline._build_loop_messages(
                context=self.context,
                enabled_tools=self.enabled_tools,
                kb_seed=seed_block,
                include_tool_manifest=bool(self.tool_schemas),
            )
            outcome = await self._run_loop(
                messages=messages,
                state=state,
                checkpoint_boundary=len(messages),
            )

        if state.sources:
            await self.stream.sources(
                state.sources,
                source="chat",
                stage=LOOP_STAGE,
                metadata={"trace_kind": "sources"},
            )
        await emit_capability_result(
            self.stream,
```

#### `_clean()` (第 221 行)


```python
        return clean_thinking_tags(text, self.pipeline.binding, self.pipeline.model).strip()
```

#### `_run_loop()` (第 226 行)
Run rounds of one LLM call + tool dispatch over *messages*.

```python
        self,
        *,
        messages: list[dict[str, Any]],
        state: AgentLoopState,
        checkpoint_boundary: int,
    ) -> LoopOutcome:
        """Run rounds of one LLM call + tool dispatch over *messages*.

        A round with tool calls keeps its assistant message (text + tool
        calls) and the ``role=tool`` results in-conversation, then continues.
        A round with no tool calls is the finish: its text — already streamed
        to the user — is the answer, and the loop ends.
        """
        explore_label = self.pipeline._t("labels.exploring", default="Exploring")
        nudged_empty_finish = False
        for _round in range(max(1, self.pipeline.effective_max_rounds(self.context))):
            try:
                result = await self._call_llm(
                    messages=messages,
                    label=explore_label,
                    call_kind="agent_loop_round",
                    trace_role="explore",
                    max_tokens=self.pipeline.loop_max_tokens,
                    tool_schemas=self.tool_schemas,
                )
            except Exception as exc:
                # A mid-loop LLM failure (timeout / transient network) must not
                # discard a turn that already gathered useful work. Salvage it
                # with a forced finish; only a failure on the very first round
                # (nothing gathered yet) propagates as before.
                if state.rounds == 0:
                    raise
                logger.warning(
                    "agent loop round failed after %d round(s); forcing finish: %s",
                    state.rounds,
                    exc,
                )
                return await self._forced_finish(messages, state, reason="error")
            state.rounds += 1
            if not result.tool_calls:
```

#### `_fold_context_checkpoint()` (第 351 行)


```python
        self,
        *,
        messages: list[dict[str, Any]],
        dispatch: DispatchOutcome,
        checkpoint_boundary: int,
    ) -> int:
        summary = _last_context_checkpoint_summary(dispatch)
        if not summary:
            return checkpoint_boundary
        prefix = messages[:checkpoint_boundary]
        prefix.append(
            {
                "role": "system",
                "content": f"[Context checkpoint]\n{summary}",
            }
        )
        messages[:] = prefix
        return len(messages)
```

#### `_forced_finish()` (第 371 行)


```python
        self,
        messages: list[dict[str, Any]],
        state: AgentLoopState,
        *,
        reason: str = "budget",
    ) -> LoopOutcome:
        if reason == "error":
            notice = self.pipeline._t(
                "notices.loop_error_finish",
                default="A step failed; answering with what has been gathered.",
            )
        else:
            notice = self.pipeline._t(
                "notices.loop_budget_exhausted",
                default="Exploration budget reached; answering with what has been gathered.",
            )
        await self.stream.progress(
            notice,
            source="chat",
            stage=LOOP_STAGE,
            metadata={"trace_kind": "warning"},
        )
        messages.append({"role": "user", "content": self.pipeline._finish_exhausted_instruction()})
        try:
            result = await self._call_llm(
                messages=messages,
                label=self.pipeline._t("labels.final_response", default="Final response"),
                call_kind="llm_final_response",
                trace_role="response",
                max_tokens=self.pipeline.loop_max_tokens,
                tool_schemas=None,  # tools disabled so the model must finish
            )
        except Exception as exc:
            # The salvage call itself failed (e.g. the provider is still
            # stalling). Don't bubble up and lose the turn — emit the graceful
            # fallback answer instead.
            logger.warning("forced-finish LLM call failed: %s", exc)
            return await self._finalize_finish("")
        state.rounds += 1
        return await self._finalize_finish(result.text)
```

#### `_finalize_finish()` (第 413 行)


```python
        final_text = self._clean(raw_text)
        if not final_text:
            # The finish round produced no usable text; nothing streamed to
            # the user, so emit a fallback answer here.
            final_text = self.pipeline._t(
                "notices.empty_final_response",
                default=(
                    "I could not produce a useful response from the model "
                    "output. Please try again or narrow the request."
                ),
            )
            await self.pipeline._emit_protocol_fallback_final_response(self.stream, final_text)
        return LoopOutcome(final_text=final_text, completed=True)
```

#### `_call_llm()` (第 430 行)


```python
        self,
        *,
        messages: list[dict[str, Any]],
        label: str,
        call_kind: str,
        trace_role: str,
        max_tokens: int,
        tool_schemas: list[dict[str, Any]] | None = None,
    ) -> LLMCallResult:
        await self.pipeline._guard_context_window(messages, self.stream)
        stage = LOOP_STAGE
        call_id = new_call_id(f"chat-{stage}")
        trace_meta = build_trace_metadata(
            call_id=call_id,
            phase=stage,
            label=label,
            call_kind=call_kind,
            trace_id=call_id,
            trace_role=trace_role,
            trace_group="stage",
        )
        await self.stream.progress(
            label,
            source="chat",
            stage=stage,
            metadata=merge_trace_metadata(
                trace_meta,
                {"trace_kind": "call_status", "call_state": "running"},
            ),
        )

        kwargs: dict[str, Any] = {
            "model": self.pipeline.model,
            "messages": messages,
            "stream": True,
            **self.pipeline._completion_kwargs(max_tokens=max_tokens),
        }
        if self.pipeline.usage is not None:
            kwargs["stream_options"] = {"include_usage": True}
        if tool_schemas:
```

#### `_create_response_stream()` (第 593 行)


```python
        self,
        kwargs: dict[str, Any],
        trace_meta: dict[str, Any],
        stage: str,
    ) -> Any:
        try:
            return await self.client.chat.completions.create(**kwargs)
        except Exception as exc:
            if "stream_options" in kwargs and is_stream_options_unsupported(exc):
                retry_kwargs = dict(kwargs)
                retry_kwargs.pop("stream_options", None)
                return await self.client.chat.completions.create(**retry_kwargs)
            if kwargs.get("tools") and is_tool_schema_unsupported(exc):
                await self.stream.progress(
                    self.pipeline._t(
                        "notices.tool_schema_fallback",
                        default="Provider rejected native tool schemas; retrying without tools.",
                    ),
                    source="chat",
                    stage=stage,
                    metadata=merge_trace_metadata(
                        trace_meta,
                        {"trace_kind": "warning", "tool_schema_fallback": True},
                    ),
                )
                retry_kwargs = dict(kwargs)
                retry_kwargs.pop("tools", None)
                retry_kwargs.pop("tool_choice", None)
                self.tool_schemas = None
                return await self.client.chat.completions.create(**retry_kwargs)
            if is_image_input_unsupported(exc) and should_degrade_to_text(
                self.pipeline.binding,
                self.pipeline.model,
                kwargs.get("messages") or [],
            ):
                strip_image_parts_inplace(kwargs["messages"])
                await self.stream.progress(
                    self.pipeline._t(
                        "notices.image_fallback",
                        default="Model does not support image input; retrying without images.",
```

## 源码级方法清单
### `deeptutor/agents/chat/agent_loop.py` (667 行)
**类 `InlineThinkFilter`**（第 64 行）> Incremental ``<think>``/``<thinking>`` splitter for streamed content.
方法：`__init__`, `feed`, `flush`, `_kind`
**类 `AgentLoopState`**（第 120 行）> Turn-level counters shared across the loop's rounds.
**类 `LLMCallResult`**（第 129 行）
**类 `LoopOutcome`**（第 136 行）> Result of running the turn's loop.
**类 `AgentLoop`**（第 149 行）> Run one chat turn as a single agent loop over one conversation.
方法：`__init__`, `run`, `_clean`, `_run_loop`, `_fold_context_checkpoint`, `_forced_finish`, `_finalize_finish`, `_call_llm`, `_create_response_stream`
**函数/方法 `_last_context_checkpoint_summary`**（第 646 行）
