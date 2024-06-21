## This is the official repository for MACC: Multi-Agent Code Completion
We propose MACC, the first lightweight multi-agent framework specifically designed for code completion. Our primary objective is to enhance code completion performance while adhering to the real-time constraints of interactive coding environments. MACC is comprised of three specialized expert agents, each focusing on distinct facets essential for code completion. In particular, the _Semantics Agent_ captures the lexical meaning of the incomplete code, the _Completion Agent_ explores potential completions, and the _Summarization Agent_ provides concise summaries of the code context. These agents leverage retrieval tools to access and extract relevant information from a vector database, which effectively functions as their memory. To improve the efficiency while minimizing computational demands, we simplify multi-agent cooperation and iterative reflection by employing contextual multi-armed bandits. This mechanism efficiently determines the most useful pieces of retrieved information at a marginal cost.
<p align="center">
<img src="samples/framework.png" alt="results" width="800" height="auto">
</p>


### Evalution
For the open-soure benchmark, use the following script:
```bash
# Download the deepseek-coder-6.7b-instruct model to your local path.
pip install -r requirements.txt
python ./evaluation/run_evaluation_vllm.py \
    --model_path path/deepseek-coder-6.7b-instruct \
    --model_name deepseek \
    --max_new_tokens 128 \
    --testset_path ./benchmark/open-source/test_macc_6.7.json \
    --total_budget 4096 \
    --sample_num -1 \
    --use_vllm True \
    --use_rag True \
    --gpus 8 \
    --gpu_memory_utilization 0.82 \
    --temperature 0 \
    --retrieval completion_agent \
    --group_key type \
    --max_rag_num 1
```

For the cceval benchmark, use the following script to build the environment:
```
git clone https://github.com/amazon-science/cceval.git ./benchmark/cceval
cd ./benchmark/cceval
pip install -r requirements.txt
pip install tree-sitter==0.21.3  # fix tree-sitter version incompatibility issues.
bash scripts/build_treesitter.sh
```
Then use of official inference and evaluation scripts, please refer to (cceval)[https://github.com/amazon-science/cceval].
```
cd path_to_macc
# for inference
python benchmark/cceval/scripts/vllm_inference.py \
    --data_root_dir ./benchmark/cceval_data \
    --tp_size 8 \
    --task line_completion_bm25 \
    --language java \
    --model path/deepseek-coder-1.3b-instruct \
    --output_dir ./tmp \
    --model_max_tokens 16384 \
    --use_crossfile_context \
    --crossfile_max_tokens 12800

# for evaluation
python benchmark/cceval/scripts/eval.py \
    --prompt_file ./benchmark/cceval_data/java/line_completion_bm25.jsonl \
    --output_dir ./tmp \
    --ts_lib ./benchmark/cceval/build/java-lang-parser.so \
    --language java \
    --only_compute_metric
```
