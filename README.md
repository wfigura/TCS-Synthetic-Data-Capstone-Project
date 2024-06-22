# TCS-Synthetic-Data-Capstone-Project

The current landscape for synthetic data generation primarily relies on domain-specific algorithms and generative adversarial networks (GANs) or large language models to produce data, namely formatted text and tabular data, that mimics real-world distributions. While effective in certain contexts, these solutions often face challenges in generating complex, multi-modal data and ensuring minimal data hallucination, which will create non-realistic or implausible data points that compromise the data quality. 

Our project introduces a reinvention using a multi-agent network large language model (MA-LLM) framework designed to address these limitations. This approach enables generating high-quality, cross-modality synthetic data with greater adherence to real-world distributions and contexts while providing maximized efficiency. By integrating multi-agent specialized in different data modalities, our platform significantly reduces the occurrence of hallucinations, ensuring that the generated synthetic data is both realistic and diverse.

---

### Setup (in the terminal)  
1. Make a local repository: `git clone https://github.com/wfigura/TCS-Synthetic-Data-Capstone-Project.git`
2. Move directory: `cd TCS-Synthetic-Data-Capstone-Project/chat-ui`
3. Install required dependencies: `npm install`
4. Run the application locally: `npm run dev`

Note: To use npm, it is required that the NodeJS package is installed. 

---

### Files
Generally, svelte files are for the front end (UI) while ts files are for the backend. The models used are in the .env file. 
