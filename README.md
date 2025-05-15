# rag-workflow

<details>
<summary>🇬🇧 English Version</summary>

### This is the English content

...

</details>

---

0.当我们搭建一个知识库时，第一步是考虑是否真的要用到 RAG。当知识库小于 200k tokens（约 500 页材料），直接将整个知识库包含在你给模型的提示中即可，无需RAG。
	
1.数据预处理
术语规范化或缩写词扩展。
	
2.Metadata Mapping 元数据映射
给原始数据加入元数据字段，json 格式，里面包含如 "source path"、"year" 等。
	
3.Chunk 切块
○ 文档结构简单的 txt，直接按固定数量的字符切分，可选的方法 text splitter。
○ 文档结构复杂的 txt，有多种分隔符如换行、空格，使用 Recursive Character Text Splitter，切分字数不固定，更灵活。
○ Document，如含有 PDF，图片，table，code（Markdown、Python、JS有点像 Recursive Character，但有不同的分隔符）：使用 Unstructured 提取数据。面对多模态（text+image），使用 GPT4-V, LLaVA, or FUYU-8b 对 image 生成 text summaries,再 embedding 后存入 vector db；或者使用 CLIP 对 image 生成 embedding。
○ 语义级的切块，有一种方法是，按句号分句，然后比较前、后句子的相似度，找到差异较大点，在此切分。
○ Agentic 智能级切分：如 LangHub 的 wfh/proposal-indexing，精心写的 prompt 来指导LLM切分。
增强表达手段：通过 padding 或者提示词工程，给chunk加入上下文。
	
4.Embedding 生成嵌入向量
后续的稠密检索的准备。 可选的 Embedding 模型 Sentence Transformer（如all-MiniLM-*、Sentence-BERT），生成后存入 Vector DB 向量数据库，如 chroma、FAISS。
	
5.Reformulate 查询重写
可选模型：seq-to-seq 架构，BART。

6.Retrieval 检索  稀疏+稠密
稀疏：基于关键词检索，也称为基于词频检索，TF-IDF，如BM25。
稠密：基于语义检索，如 FAISS 建立索引，计算 Embedding后的 query 和 文档的 L2距离。
也可以，cosine similarity search 余弦相似度，可以计算 TF-IDF 向量 或 词嵌入 的相似性。
 
7.Rerank 重排
○ MMR（Maximal Marginal Relevance，最大边界相关性）：保证相关性的同时，提高多样性。
○ 将一对输入（query和初次检索到的文档），使用 cross-encoder (交叉编码器，如ms-marco-MiniLM-*) ，计算它们的相关性或相似性得分。这种模型速度比较慢，但比 BM25 和L2 距离更精确。
 
8.Filter 过滤 
基于 metadata 元数据，重排 或者 过滤不符合 query 的，比如从大到小、年龄>18、出厂日期>20250515。
 
9.Eval 评估 
○ 评估 RAG 中“R”的有效性，检索质量：1命中率、2平均倒数排名、3精准率、4召回率、5NDCG。
○ 再是“G”生成质量：1忠实度、2答案相关性、3上下文相关性、4答案正确性、5流畅性、6简洁性、7无害性。
○ 端到端系统性能：1用户满意度、2任务完成率、3无法回答率、4响应时间/延迟、5吞吐量。
○ 运营和成本指标：1计算成本、2数据更新与维护成本、3可扩展性、4鲁棒性。
评估策略和工具：人工、黄金数据集、RAGAS、AB 测试、LLM。
