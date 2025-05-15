# rag-workflow

The content of the English Version is from the translation using Gemini.

<details>
<summary> English Version</summary>

### This is the English Version

0.When building a knowledge base, the first step is to consider whether RAG is truly necessary. If the knowledge base is smaller than 200k tokens (approximately 500 pages of material), you can directly include the entire knowledge base in the prompt given to the model, without needing RAG.

1.Data Preprocessing<br>
Terminology normalization or acronym expansion.

2.Metadata Mapping<br>
Add metadata fields to the raw data, in JSON format, including fields like "source path," "year," etc.

3.Chunking<br>
For TXT files with simple document structures, directly split by a fixed number of characters; an optional method is using a text splitter.
For TXT files with complex document structures and multiple delimiters like newlines or spaces, use Recursive Character Text Splitter, which offers more flexible, non-fixed size splitting.
For Documents, such as those containing PDFs, images, tables, or code (Markdown, Python, JS are somewhat similar to Recursive Character splitting but with different delimiters): use Unstructured to extract data. For multimodal data (text+image), use models like GPT-4V, LLaVA, or FUYU-8b to generate text summaries for images, then embed and store them in a vector DB; alternatively, use CLIP to generate image embeddings.
For semantic-level chunking, one method is to split by sentences (e.g., at periods), then compare the similarity of consecutive sentences to find points of significant difference and split there.
Agentic chunking (intelligent splitting): For example, LangHub's wfh/proposal-indexing uses carefully crafted prompts to guide an LLM in the splitting process.
Enhancing expressiveness: Add context to chunks through padding or prompt engineering.

4.Embedding / Generating Embeddings<br>
Preparation for subsequent dense retrieval. Optional Embedding models: Sentence Transformers (e.g., all-MiniLM-*, Sentence-BERT). After generation, store in a Vector DB (vector database) like Chroma or FAISS.

5.Query Reformulation<br>
Optional models: seq-to-seq architecture, BART.

6.Retrieval: Sparse + Dense<br>
Sparse: Keyword-based retrieval, also known as term frequency-based retrieval, TF-IDF, e.g., BM25.
Dense: Semantic-based retrieval, e.g., building an index with FAISS and calculating the L2 distance between the embeddings of the query and documents.
Alternatively, cosine similarity search can be used to calculate the similarity of TF-IDF vectors or word embeddings.

7.Reranking<br>
MMR (Maximal Marginal Relevance): Ensures relevance while increasing diversity.
Use a cross-encoder (e.g., ms-marco-MiniLM-*) with a pair of inputs (query and initially retrieved document) to calculate their relevance or similarity score. These models are slower but more accurate than BM25 and L2 distance.

8.Filtering<br>
Based on metadata, rerank or filter out results that do not conform to the query, e.g., sort descending, age > 18, production date > 20250515.

9.Evaluation<br>
Evaluate the effectiveness of "R" in RAG (Retrieval Quality): 1. Hit Rate, 2. Mean Reciprocal Rank (MRR), 3. Precision, 4. Recall, 5. NDCG (Normalized Discounted Cumulative Gain).
Then, "G" (Generation Quality): 1. Faithfulness, 2. Answer Relevance, 3. Context Relevance/Utilization, 4. Answer Correctness, 5. Fluency, 6. Conciseness, 7. Harmlessness.
End-to-End System Performance: 1. User Satisfaction, 2. Task Completion Rate, 3. No Answer Rate / Rejection Rate, 4. Response Time / Latency, 5. Throughput.
Operational and Cost Metrics: 1. Computational Cost, 2. Data Update & Maintenance Cost, 3. Scalability, 4. Robustness.
Evaluation Strategies and Tools: Human evaluation, Golden Datasets, RAGAS, A/B Testing, LLM-as-a-judge

</details>

---

0.当我们搭建一个知识库时，第一步是考虑是否真的要用到 RAG。当知识库小于 200k tokens（约 500 页材料），直接将整个知识库包含在你给模型的提示中即可，无需RAG。
	
1.数据预处理<br>
术语规范化或缩写词扩展。
	
2.Metadata Mapping 元数据映射<br>
给原始数据加入元数据字段，json 格式，里面包含如 "source path"、"year" 等。
	
3.Chunk 切块<br>
○ 文档结构简单的 txt，直接按固定数量的字符切分，可选的方法 text splitter。
○ 文档结构复杂的 txt，有多种分隔符如换行、空格，使用 Recursive Character Text Splitter，切分字数不固定，更灵活。
○ Document，如含有 PDF，图片，table，code（Markdown、Python、JS有点像 Recursive Character，但有不同的分隔符）：使用 Unstructured 提取数据。面对多模态（text+image），使用 GPT4-V, LLaVA, or FUYU-8b 对 image 生成 text summaries,再 embedding 后存入 vector db；或者使用 CLIP 对 image 生成 embedding。
○ 语义级的切块，有一种方法是，按句号分句，然后比较前、后句子的相似度，找到差异较大点，在此切分。
○ Agentic 智能级切分：如 LangHub 的 wfh/proposal-indexing，精心写的 prompt 来指导LLM切分。
增强表达手段：通过 padding 或者提示词工程，给chunk加入上下文。
	
4.Embedding 生成嵌入向量<br>
后续的稠密检索的准备。 可选的 Embedding 模型 Sentence Transformer（如all-MiniLM-*、Sentence-BERT），生成后存入 Vector DB 向量数据库，如 chroma、FAISS。
	
5.Reformulate 查询重写<br>
可选模型：seq-to-seq 架构，BART。

6.Retrieval 检索  稀疏+稠密<br>
稀疏：基于关键词检索，也称为基于词频检索，TF-IDF，如BM25。
稠密：基于语义检索，如 FAISS 建立索引，计算 Embedding后的 query 和 文档的 L2距离。
也可以，cosine similarity search 余弦相似度，可以计算 TF-IDF 向量 或 词嵌入 的相似性。
 
7.Rerank 重排<br>
○ MMR（Maximal Marginal Relevance，最大边界相关性）：保证相关性的同时，提高多样性。
○ 将一对输入（query和初次检索到的文档），使用 cross-encoder (交叉编码器，如ms-marco-MiniLM-*) ，计算它们的相关性或相似性得分。这种模型速度比较慢，但比 BM25 和L2 距离更精确。
 
8.Filter 过滤<br>
基于 metadata 元数据，重排 或者 过滤不符合 query 的，比如从大到小、年龄>18、出厂日期>20250515。
 
9.Eval 评估<br>
○ 评估 RAG 中“R”的有效性，检索质量：1命中率、2平均倒数排名、3精准率、4召回率、5NDCG。
○ 再是“G”生成质量：1忠实度、2答案相关性、3上下文相关性、4答案正确性、5流畅性、6简洁性、7无害性。
○ 端到端系统性能：1用户满意度、2任务完成率、3无法回答率、4响应时间/延迟、5吞吐量。
○ 运营和成本指标：1计算成本、2数据更新与维护成本、3可扩展性、4鲁棒性。
评估策略和工具：人工、黄金数据集、RAGAS、AB 测试、LLM。
