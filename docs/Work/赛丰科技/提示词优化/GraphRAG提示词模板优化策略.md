---
date created: 2025/5/7 8:54
date modified: 2025/5/7 10:22
---
# 自动提示调优 ⚙️

GraphRAG 提供了为知识图谱生成创建领域自适应提示的功能。虽然这一步是可选的，但强烈建议运行它，因为在执行索引运行时将获得更好的结果。

**提示的生成过程包括加载输入数据，将其拆分为块（文本单元），然后通过多次调用 LLM 并进行模板替换，生成最终的提示**。我们建议使用脚本提供的默认值，但如果您希望深入探索和调整提示调优算法，可以参考本页面中的详细设置。

![[IMG-2025-05-07-10-17-50.png]]

## 前置条件

在运行自动调优之前，请确保您已使用 `graphrag init` 命令初始化工作空间。这将创建必要的配置文件和默认提示。有关初始化过程的更多信息，请参阅 [初始化文档](https://chatgpt.com/config/init.md)。

## 使用方法

可以通过命令行运行主脚本，并使用各种选项：

```bash
graphrag prompt-tune [--root ROOT] [--config CONFIG] [--domain DOMAIN] [--selection-method METHOD] [--limit LIMIT] [--language LANGUAGE] \
[--max-tokens MAX_TOKENS] [--chunk-size CHUNK_SIZE] [--n-subset-max N_SUBSET_MAX] [--k K] \
[--min-examples-required MIN_EXAMPLES_REQUIRED] [--discover-entity-types] [--output OUTPUT]
```

## 命令行选项

- `--config`（必需）：配置文件的路径，用于加载数据和模型设置。
- `--root`（可选）：数据项目的根目录，包括配置文件（YML、JSON 或 .env）。默认为当前目录。
- `--domain`（可选）：与输入数据相关的领域，例如“空间科学”、“微生物学”或“环境新闻”。如果留空，领域将从输入数据中自动推断。
- `--selection-method`（可选）：选择文档的方法。可选值包括 `all`、`random`、`auto` 或 `top`，默认为 `random`。
- `--limit`（可选）：在使用 `random` 或 `top` 选择方法时加载的文本单元数量限制。默认为 15。
- `--language`（可选）：用于输入处理的语言。如果与输入的语言不同，LLM 会自动翻译。默认值为""，表示将自动检测输入的语言。
- `--max-tokens`（可选）：生成提示的最大 token 数。默认为 2000。
- `--chunk-size`（可选）：从输入文档生成文本单元时使用的 token 数。默认为 200。
- `--n-subset-max`（可选）：在使用 `auto` 选择方法时嵌入的文本块数量。默认为 300。
- `--k`（可选）：在使用 `auto` 选择方法时选择的文档数量。默认为 15。
- `--min-examples-required`（可选）：进行实体抽取提示时所需的最小示例数量。默认为 2。
- `--discover-entity-types`（可选）：允许 LLM 自动发现并抽取实体。建议在数据涵盖多个主题或高度随机化时使用。
- `--output`（可选）：用于保存生成的提示的文件夹。默认为 `prompts`。

## 示例用法

```bash
python -m graphrag prompt-tune --root /path/to/project --config /path/to/settings.yaml --domain "environmental news" \
--selection-method random --limit 10 --language English --max-tokens 2048 --chunk-size 256 --min-examples-required 3 \
--no-entity-types --output /path/to/output
```

或者使用最小配置（推荐）：

```bash
python -m graphrag prompt-tune --root /path/to/project --config /path/to/settings.yaml --no-entity-types
```

## 文档选择方法

自动调优功能会读取输入数据，然后根据 `chunk-size` 参数将其划分为文本单元。随后，使用以下方法之一选择用于提示生成的文本样本：

- **random**：随机选择文本单元。这是默认且推荐的选项。
- **top**：选择前 n 个文本单元。
- **all**：使用所有文本单元进行生成。仅在小型数据集时使用，通常不推荐。
- **auto**：将文本单元嵌入到低维空间，并选择与质心最接近的 k 个邻居。当数据集较大且需要选择具有代表性的样本时非常有用。

## 修改环境变量

在运行自动调优后，应修改以下环境变量（或配置变量）以确保在执行索引运行时能够使用新生成的提示。注意：请确保更新到生成提示的正确路径，在本示例中使用的是默认的 `prompts` 路径。

```bash
GRAPHRAG_ENTITY_EXTRACTION_PROMPT_FILE = "prompts/entity_extraction.txt"

GRAPHRAG_COMMUNITY_REPORT_PROMPT_FILE = "prompts/community_report.txt"

GRAPHRAG_SUMMARIZE_DESCRIPTIONS_PROMPT_FILE = "prompts/summarize_descriptions.txt"
```

或者在 `yaml` 配置文件中：

```yaml
entity_extraction:
  prompt: "prompts/entity_extraction.txt"

summarize_descriptions:
  prompt: "prompts/summarize_descriptions.txt"

community_reports:
  prompt: "prompts/community_report.txt"
```

---

## Python API

```python
# Copyright (c) 2024 Microsoft Corporation.
# Licensed under the MIT License

"""
Auto Templating API.

This API provides access to the auto templating feature of graphrag, allowing external applications
to hook into graphrag and generate prompts from private data.

WARNING: This API is under development and may undergo changes in future releases.
Backwards compatibility is not guaranteed at this time.
"""

from typing import Annotated

import annotated_types
from pydantic import PositiveInt, validate_call

from graphrag.callbacks.noop_workflow_callbacks import NoopWorkflowCallbacks
from graphrag.config.defaults import graphrag_config_defaults, language_model_defaults
from graphrag.config.models.graph_rag_config import GraphRagConfig
from graphrag.language_model.manager import ModelManager
from graphrag.logger.base import ProgressLogger
from graphrag.prompt_tune.defaults import MAX_TOKEN_COUNT, PROMPT_TUNING_MODEL_ID
from graphrag.prompt_tune.generator.community_report_rating import (
    generate_community_report_rating,
)
from graphrag.prompt_tune.generator.community_report_summarization import (
    create_community_summarization_prompt,
)
from graphrag.prompt_tune.generator.community_reporter_role import (
    generate_community_reporter_role,
)
from graphrag.prompt_tune.generator.domain import generate_domain
from graphrag.prompt_tune.generator.entity_relationship import (
    generate_entity_relationship_examples,
)
from graphrag.prompt_tune.generator.entity_summarization_prompt import (
    create_entity_summarization_prompt,
)
from graphrag.prompt_tune.generator.entity_types import generate_entity_types
from graphrag.prompt_tune.generator.extract_graph_prompt import (
    create_extract_graph_prompt,
)
from graphrag.prompt_tune.generator.language import detect_language
from graphrag.prompt_tune.generator.persona import generate_persona
from graphrag.prompt_tune.loader.input import load_docs_in_chunks
from graphrag.prompt_tune.types import DocSelectionType


@validate_call(config={"arbitrary_types_allowed": True})
async def generate_indexing_prompts(
    config: GraphRagConfig,
    logger: ProgressLogger,
    root: str,
    chunk_size: PositiveInt = graphrag_config_defaults.chunks.size,
    overlap: Annotated[
        int, annotated_types.Gt(-1)
    ] = graphrag_config_defaults.chunks.overlap,
    limit: PositiveInt = 15,
    selection_method: DocSelectionType = DocSelectionType.RANDOM,
    domain: str | None = None,
    language: str | None = None,
    max_tokens: int = MAX_TOKEN_COUNT,
    discover_entity_types: bool = True,
    min_examples_required: PositiveInt = 2,
    n_subset_max: PositiveInt = 300,
    k: PositiveInt = 15,
) -> tuple[str, str, str]:
    """Generate indexing prompts.

    Parameters
    ----------
    - config: The GraphRag configuration.
    - logger: The logger to use for progress updates.
    - root: The root directory.
    - output_path: The path to store the prompts.
    - chunk_size: The chunk token size to use for input text units.
    - limit: The limit of chunks to load.
    - selection_method: The chunk selection method.
    - domain: The domain to map the input documents to.
    - language: The language to use for the prompts.
    - max_tokens: The maximum number of tokens to use on entity extraction prompts
    - discover_entity_types: Generate entity types.
    - min_examples_required: The minimum number of examples required for entity extraction prompts.
    - n_subset_max: The number of text chunks to embed when using auto selection method.
    - k: The number of documents to select when using auto selection method.

    Returns
    -------
    tuple[str, str, str]: entity extraction prompt, entity summarization prompt, community summarization prompt
    """
    # Retrieve documents
    logger.info("Chunking documents...")
    doc_list = await load_docs_in_chunks(
        root=root,
        config=config,
        limit=limit,
        select_method=selection_method,
        logger=logger,
        chunk_size=chunk_size,
        overlap=overlap,
        n_subset_max=n_subset_max,
        k=k,
    )

    # Create LLM from config
    # TODO: Expose a way to specify Prompt Tuning model ID through config
    logger.info("Retrieving language model configuration...")
    default_llm_settings = config.get_language_model_config(PROMPT_TUNING_MODEL_ID)

    # if max_retries is not set, inject a dynamically assigned value based on the number of expected LLM calls
    # to be made or fallback to a default value in the worst case
    if default_llm_settings.max_retries < -1:
        default_llm_settings.max_retries = min(
            len(doc_list), language_model_defaults.max_retries
        )
        msg = f"max_retries not set, using default value: {default_llm_settings.max_retries}"
        logger.warning(msg)

    logger.info("Creating language model...")
    llm = ModelManager().register_chat(
        name="prompt_tuning",
        model_type=default_llm_settings.type,
        config=default_llm_settings,
        callbacks=NoopWorkflowCallbacks(),
        cache=None,
    )

    if not domain:
        logger.info("Generating domain...")
        domain = await generate_domain(llm, doc_list)

    if not language:
        logger.info("Detecting language...")
        language = await detect_language(llm, doc_list)

    logger.info("Generating persona...")
    persona = await generate_persona(llm, domain)

    logger.info("Generating community report ranking description...")
    community_report_ranking = await generate_community_report_rating(
        llm, domain=domain, persona=persona, docs=doc_list
    )

    entity_types = None
    extract_graph_llm_settings = config.get_language_model_config(
        config.extract_graph.model_id
    )
    if discover_entity_types:
        logger.info("Generating entity types...")
        entity_types = await generate_entity_types(
            llm,
            domain=domain,
            persona=persona,
            docs=doc_list,
            json_mode=extract_graph_llm_settings.model_supports_json or False,
        )

    logger.info("Generating entity relationship examples...")
    examples = await generate_entity_relationship_examples(
        llm,
        persona=persona,
        entity_types=entity_types,
        docs=doc_list,
        language=language,
        json_mode=False,  # config.llm.model_supports_json should be used, but these prompts are used in non-json mode by the index engine
    )

    logger.info("Generating entity extraction prompt...")
    extract_graph_prompt = create_extract_graph_prompt(
        entity_types=entity_types,
        docs=doc_list,
        examples=examples,
        language=language,
        json_mode=False,  # config.llm.model_supports_json should be used, but these prompts are used in non-json mode by the index engine
        encoding_model=extract_graph_llm_settings.encoding_model,
        max_token_count=max_tokens,
        min_examples_required=min_examples_required,
    )

    logger.info("Generating entity summarization prompt...")
    entity_summarization_prompt = create_entity_summarization_prompt(
        persona=persona,
        language=language,
    )

    logger.info("Generating community reporter role...")
    community_reporter_role = await generate_community_reporter_role(
        llm, domain=domain, persona=persona, docs=doc_list
    )

    logger.info("Generating community summarization prompt...")
    community_summarization_prompt = create_community_summarization_prompt(
        persona=persona,
        role=community_reporter_role,
        report_rating_description=community_report_ranking,
        language=language,
    )

    logger.info(f"\nGenerated domain: {domain}")  # noqa: G004
    logger.info(f"\nDetected language: {language}")  # noqa: G004
    logger.info(f"\nGenerated persona: {persona}")  # noqa: G004

    return (
        extract_graph_prompt,
        entity_summarization_prompt,
        community_summarization_prompt,
    )
```