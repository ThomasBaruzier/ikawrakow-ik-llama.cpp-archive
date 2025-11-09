## 🗣️ [Discussion #715](https://github.com/ikawrakow/ik_llama.cpp/discussions/715) - Perplexity vs Size Graphs for the recent quants (Deepseek-V3.1-Terminus, Deepseek-R1, Qwen3-Coder, Kimi-K2, Chimera etc.)

| **Author** | `magikRUKKOLA` |
| :--- | :--- |
| **State** | ✅ **Open** |
| **Created** | 2025-08-21 |
| **Updated** | 2025-10-31 |

---

## 📄 Description

GRAPHS:

![v31-terminus-ppl-log](https://github.com/user-attachments/assets/bd2603f0-e17c-4208-90fe-1da074a7fc05)
![r1-0528-ppl-log](https://github.com/user-attachments/assets/fa741f06-2a2e-4e80-be7a-cea7363c9c97)
![v31-ppl-log](https://github.com/user-attachments/assets/ba59ed83-0144-4f90-ba21-0eaa12f3f965)
![chimera-ppl-log](https://github.com/user-attachments/assets/2f71ba38-f620-4571-a4c7-dfcc8603b89d)
![kimi-0905-log-ppl](https://github.com/user-attachments/assets/4b099eb1-825f-41ee-906c-b8075471f0e2)
![glm46-ppl-log](https://github.com/user-attachments/assets/dfba245d-7b42-4944-ab89-48208e3c34cd)
![qwen3-coder-ppl-log](https://github.com/user-attachments/assets/da6ac751-8618-4794-ae30-a404e82c299e)




DATA SOURCES:

```json
{
  "title": "DeepSeek-V3.1-Terminus (671B) Quantization Analysis",
  "subtitle": "Lower perplexity = Better performance",
  "model_parameters": 671000000000,
  "data": [
    {"name": "IQ1_S", "bpw": 1.745, "ppl": 5.4829, "size": 134.45, "url": "https://huggingface.co/ubergarm/DeepSeek-V3.1-Terminus-GGUF/tree/main/IQ1_S"},
    {"name": "IQ1_KT", "bpw": 1.987, "ppl": 4.5310, "size": 154.61, "url": "https://huggingface.co/ubergarm/DeepSeek-V3.1-Terminus-GGUF/tree/main/IQ1_KT"},
    {"name": "IQ2_KS", "bpw": 2.472, "ppl": 4.0280, "size": 190.56, "url": "https://huggingface.co/ubergarm/DeepSeek-V3.1-Terminus-GGUF/tree/main/IQ2_KS"},
    {"name": "IQ2_KL", "bpw": 2.962, "ppl": 3.7112, "size": 228.54, "url": "https://huggingface.co/ubergarm/DeepSeek-V3.1-Terminus-GGUF/tree/main/IQ2_KL"},
    {"name": "IQ3_KS", "bpw": 3.545, "ppl": 3.5174, "size": 276.89, "url": "https://huggingface.co/ubergarm/DeepSeek-V3.1-Terminus-GGUF/tree/main/IQ3_KS"},
    {"name": "IQ3_K", "bpw": 3.724, "ppl": 3.4781, "size": 290.56, "url": "https://huggingface.co/ubergarm/DeepSeek-V3.1-Terminus-GGUF/tree/main/IQ3_K"},
    {"name": "smol-IQ4_KSS", "bpw": 4.080, "ppl": 3.4445, "size": 317.96, "url": "https://huggingface.co/ubergarm/DeepSeek-V3.1-Terminus-GGUF/tree/main/smol-IQ4_KSS"},
    {"name": "IQ4_K", "bpw": 4.896, "ppl": 3.4198, "size": 380.57, "url": "https://huggingface.co/ubergarm/DeepSeek-V3.1-Terminus-GGUF/tree/main/IQ4_K"},
    {"name": "smol-IQ5_KS", "bpw": 5.339, "ppl": 3.4059, "size": 416.27, "url": "https://huggingface.co/ubergarm/DeepSeek-V3.1-Terminus-GGUF/tree/main/smol-IQ5_KS"},
    {"name": "THIREUS-5.4498bpw-R4", "bpw": 5.4498, "ppl": 3.3961, "size": 426.07, "url": "https://github.com/ikawrakow/ik_llama.cpp/discussions/715#discussioncomment-14579570"},
    {"name": "IQ5_K", "bpw": 5.941, "ppl": 3.4000, "size": 462.87, "url": "https://huggingface.co/ubergarm/DeepSeek-V3.1-Terminus-GGUF/tree/main/IQ5_K"},
    {"name": "THIREUS-6.2212bpw", "bpw": 6.2212, "ppl": 3.3949, "size": 485.07, "url": "https://github.com/ikawrakow/ik_llama.cpp/discussions/715#discussioncomment-14554951"},
    {"name": "Q8_0", "bpw": 8.504, "ppl": 3.3929, "size": 660.30, "url": "https://huggingface.co/ubergarm/DeepSeek-V3.1-Terminus-GGUF/tree/main/Q8_0"}
  ]
}
{
  "title": "DeepSeek-R1-0528 (671B) Quantization Analysis",
  "subtitle": "Lower perplexity = Better performance",
  "model_parameters": 671000000000,
  "data": [
    {"name": "IQ1_S_R4", "bpw": 1.664, "ppl": 4.8831, "size": 129.53, "url": "https://huggingface.co/ubergarm/DeepSeek-R1-0528-GGUF/tree/main/IQ1_S_R4"},
    {"name": "THIREUS-1.9364", "bpw": 1.9364, "ppl": 4.3533, "size": 150.75, "url": "https://github.com/Thireus/GGUF-Tool-Suite/blob/main/recipe_examples/DeepSeek-R1-0528.THIREUS-1.9364bpw-4.3533ppl.151GB-GGUF_14GB-GPU_203GB-CPU.3c88ec6_9fd615d.recipe"},
    {"name": "IQ2_KT", "bpw": 2.514, "ppl": 3.6378, "size": 197.70, "url": null},
    {"name": "THIREUS-2.7840", "bpw": 2.7840, "ppl": 3.4341, "size": 216.55, "url": "https://github.com/Thireus/GGUF-Tool-Suite/blob/main/recipe_examples/DeepSeek-R1-0528.THIREUS-2.7840bpw-3.4341ppl.217GB-GGUF_14GB-GPU_203GB-CPU.3c88ec6_02247be.recipe"},
    {"name": "IQ2_K_R4", "bpw": 2.799, "ppl": 3.5069, "size": 217.76, "url": "https://huggingface.co/ubergarm/DeepSeek-R1-0528-GGUF/tree/main/IQ2_K_R4"},
    {"name": "JWNoctis/R1-0528/IQ2_KL", "bpw": 2.930, "ppl": 3.4379, "size": 227.64, "url": "https://forum.level1techs.com/t/deepseek-deep-dive-r1-at-home/225826/354"},
    {"name": "UD_Q2_K_XL", "bpw": 2.994, "ppl": 3.5278, "size": 232.02, "url": "https://huggingface.co/unsloth/DeepSeek-R1-0528-GGUF/tree/main/UD-Q2_K_XL"},
    {"name": "THIREUS-3.1027", "bpw": 3.1027, "ppl": 3.3372, "size": 240.96, "url": "https://github.com/Thireus/GGUF-Tool-Suite/blob/main/recipe_examples/DeepSeek-R1-0528.THIREUS-3.1027bpw-3.3372ppl.242GB-GGUF_11GB-GPU_231GB-CPU.3c88ec6_adc8101.recipe"},
    {"name": "THIREUS-3.1446", "bpw": 3.1446, "ppl": 3.3257, "size": 244.39, "url": "https://github.com/Thireus/GGUF-Tool-Suite/blob/main/recipe_examples/DeepSeek-R1-0528.THIREUS-3.1446bpw-3.3257ppl.246GB-GGUF_15GB-GPU_231GB-CPU.3c88ec6_7d1efe1.recipe"},
    {"name": "THIREUS-3.1447", "bpw": 3.1447, "ppl": 3.3269, "size": 244.40, "url": "https://github.com/Thireus/GGUF-Tool-Suite/blob/main/recipe_examples/DeepSeek-R1-0528.THIREUS-3.1447bpw-3.3269ppl.246GB-GGUF_15GB-GPU_231GB-CPU.3c88ec6_4b1254a.recipe"},
    {"name": "THIREUS-3.1525", "bpw": 3.1525, "ppl": 3.3251, "size": 245.07, "url": "https://github.com/Thireus/GGUF-Tool-Suite/blob/main/recipe_examples/DeepSeek-R1-0528.THIREUS-3.1525bpw-3.3251ppl.246GB-GGUF_15GB-GPU_231GB-CPU.3c88ec6_5a3fc0f.recipe"},
    {"name": "THIREUS-3.1740", "bpw": 3.1740, "ppl": 3.3253, "size": 246.76, "url": "https://github.com/Thireus/GGUF-Tool-Suite/blob/main/recipe_examples/DeepSeek-R1-0528.THIREUS-3.1740bpw-3.3253ppl.248GB-GGUF_17GB-GPU_231GB-CPU.3c88ec6_6cf3a72.recipe"},
    {"name": "THIREUS-3.1858", "bpw": 3.1858, "ppl": 3.3261, "size": 247.60, "url": "https://github.com/Thireus/GGUF-Tool-Suite/blob/main/recipe_examples/DeepSeek-R1-0528.THIREUS-3.1858bpw-3.3261ppl.249GB-GGUF_18GB-GPU_231GB-CPU.3c88ec6_027b7ff.recipe"},
    {"name": "THIREUS-3.2564", "bpw": 3.2564, "ppl": 3.2985, "size": 253.18, "url": "https://github.com/Thireus/GGUF-Tool-Suite/blob/main/recipe_examples/DeepSeek-R1-0528.THIREUS-3.2564bpw-3.2985ppl.254GB-GGUF_15GB-GPU_239GB-CPU.3c88ec6_7c0be1e.recipe"},
    {"name": "IQ3_KT", "bpw": 3.483, "ppl": 3.3056, "size": 267.63, "url": "https://huggingface.co/ubergarm/DeepSeek-R1-0528-GGUF/tree/main/IQ3_KT"},
    {"name": "THIREUS-3.5652", "bpw": 3.5652, "ppl": 3.2734, "size": 284.90, "url": "https://github.com/Thireus/GGUF-Tool-Suite/blob/main/recipe_examples/DeepSeek-R1-0528.THIREUS-3.5652bpw-3.2734ppl.278GB-GGUF_14GB-GPU_264GB-CPU.3c88ec6_9b5660b.recipe"},
    {"name": "IQ3_KS", "bpw": 3.598, "ppl": 3.2991, "size": 287.54, "url": "https://huggingface.co/ubergarm/DeepSeek-R1-0528-GGUF/tree/main/IQ3_KS"},
    {"name": "THIREUS-3.6766", "bpw": 3.6766, "ppl": 3.2741, "size": 293.80, "url": "https://github.com/ikawrakow/ik_llama.cpp/discussions/477#discussioncomment-13781700"},
    {"name": "IQ3_K_R4", "bpw": 3.847, "ppl": 3.2730, "size": 306.52, "url": "https://huggingface.co/ubergarm/DeepSeek-R1-0528-GGUF/tree/main/IQ3_K_R4"},
    {"name": "THIREUS-3.976", "bpw": 3.976, "ppl": 3.2452, "size": 315.18, "url": "https://github.com/ikawrakow/ik_llama.cpp/discussions/477#discussioncomment-13798329"},
    {"name": "IQ4_XS (unsloth)", "bpw": 4.2683, "ppl": 3.2598, "size": 337.03, "url": "https://huggingface.co/unsloth/DeepSeek-R1-0528-GGUF/tree/main/IQ4_XS"},
    {"name": "q4_0", "bpw": 4.508, "ppl": 3.2895, "size": 356.27, "url": "https://huggingface.co/unsloth/DeepSeek-R1-0528-GGUF/tree/main/Q4_0"},
    {"name": "UD_Q4_K_XL", "bpw": 4.578, "ppl": 3.2483, "size": 361.92, "url": "https://huggingface.co/unsloth/DeepSeek-R1-0528-GGUF/tree/main/UD-Q4_K_XL"},
    {"name": "IQ4_KS_R4", "bpw": 4.701, "ppl": 3.2286, "size": 371.94, "url": "https://huggingface.co/ubergarm/DeepSeek-R1-0528-GGUF/tree/main/IQ4_KS_R4"},
    {"name":"THIREUS-5.0601","bpw":5.0601,"ppl":3.2223,"size": 397.12, "url":"https://github.com/ikawrakow/ik_llama.cpp/discussions/715#discussioncomment-14625973"},
    {"name": "DQ4_K_R4", "bpw": 5.289, "ppl": 3.2276, "size": 415.04, "url": "https://huggingface.co/anikifoss/DeepSeek-R1-0528-DQ4_K_R4"},
    {"name": "THIREUS-6.2218", "bpw": 6.2218, "ppl": 3.2240, "size": 486.97, "url": "https://github.com/ikawrakow/ik_llama.cpp/discussions/477#discussioncomment-13781560"},
    {"name": "THIREUS-6.4296", "bpw": 6.4296, "ppl": 3.2231, "size": 503.65, "url": "https://github.com/ikawrakow/ik_llama.cpp/discussions/718#discussioncomment-14193821"},
    {"name": "THIREUS-6.5522", "bpw": 6.5522, "ppl": 3.2227, "size": 512.60, "url": "https://github.com/ikawrakow/ik_llama.cpp/discussions/718#discussioncomment-14193821"},
    {"name": "Q8_0", "bpw": 8.5259260, "ppl": 3.2130, "size": 664.33, "url": "https://huggingface.co/unsloth/DeepSeek-R1-0528-GGUF/tree/main/Q8_0"}
  ]
}
{
  "title": "DeepSeek-V3.1 (671B) Quantization Analysis",
  "subtitle": "Lower perplexity = Better performance",
  "model_parameters": 671000000000,
  "data": [
    { "name": "IQ1_S", "bpw": 1.710, "ppl": 5.3113, "size": 132.84, "url": "https://huggingface.co/ubergarm/DeepSeek-V3.1-GGUF/tree/main/IQ1_S" },
    { "name": "IQ1_KT", "bpw": 1.984, "ppl": 4.3987, "size": 153.62, "url": "https://huggingface.co/ubergarm/DeepSeek-V3.1-GGUF/tree/main/IQ1_KT" },
    { "name": "IQ2_KS", "bpw": 2.472, "ppl": 3.9583, "size": 190.56, "url": "https://huggingface.co/ubergarm/DeepSeek-V3.1-GGUF/tree/main/IQ2_KS" },
    { "name": "IQ2_KT", "bpw": 2.619, "ppl": 3.8109, "size": 201.47, "url": "" },
    { "name": "IQ2_KL", "bpw": 2.960, "ppl": 3.6312, "size": 225.58, "url": "https://huggingface.co/ubergarm/DeepSeek-V3.1-GGUF/tree/main/IQ2_KL" },
    { "name": "IQ3_KS", "bpw": 3.551, "ppl": 3.4534, "size": 273.06, "url": "https://huggingface.co/ubergarm/DeepSeek-V3.1-GGUF/tree/main/IQ3_KS" },
    { "name": "IQ3_K", "bpw": 3.753, "ppl": 3.4260, "size": 287.09, "url": "https://huggingface.co/ubergarm/DeepSeek-V3.1-GGUF/tree/main/IQ3_K" },
    { "name": "smol-IQ4_KSS", "bpw": 4.080, "ppl": 3.3898, "size": 317.96, "url": "" },
    { "name": "IQ4_KSS", "bpw": 4.162, "ppl": 3.3887, "size": 325.03, "url": "https://huggingface.co/ubergarm/DeepSeek-V3.1-GGUF/tree/main/IQ4_KSS" },
    { "name": "Q4_0", "bpw": 4.507, "ppl": 3.4277, "size": 355.86, "url": "" },
    { "name": "UD-Q4_K_XL", "bpw": 4.507, "ppl": 3.4013, "size": 355.86, "url": "https://huggingface.co/unsloth/DeepSeek-V3.1-GGUF/tree/main/UD-Q4_K_XL" },
    { "name": "IQ4_KS", "bpw": 4.649, "ppl": 3.3806, "size": 367.52, "url": "https://huggingface.co/ubergarm/DeepSeek-V3.1-GGUF/tree/main/IQ4_KS" },
    { "name": "IQ4_K", "bpw": 4.925, "ppl": 3.3715, "size": 389.94, "url": "https://huggingface.co/ubergarm/DeepSeek-V3.1-GGUF/tree/main/IQ4_K" },
    { "name": "IQ5_K", "bpw": 5.944, "ppl": 3.3550, "size": 462.98, "url": "https://huggingface.co/ubergarm/DeepSeek-V3.1-GGUF/tree/main/IQ5_K" },
    { "name": "Q8_0", "bpw": 8.504, "ppl": 3.3473, "size": 660.30, "url": "https://huggingface.co/ubergarm/DeepSeek-V3.1-GGUF/tree/main/Q8_0" },
    { "name": "BF16", "bpw": 16.003, "ppl": 3.3469, "size": 1257.33, "url": "" }
  ]
}
{
  "title": "DeepSeek-TNG-R1T2-Chimera (671B) Quantization Analysis",
  "subtitle": "Lower perplexity = Better performance",
  "model_parameters": 671000000000,
  "data": [
    {"name": "IQ1_S", "bpw": 1.699, "ppl": 4.9878, "size": 132.25, "url": "https://huggingface.co/ubergarm/DeepSeek-TNG-R1T2-Chimera-GGUF/tree/main/IQ1_S"},
    {"name": "THIREUS-1.6693", "bpw": 1.6693, "ppl": 4.9676, "size": 130.58, "url": "https://github.com/ikawrakow/ik_llama.cpp/discussions/477#discussioncomment-13883488"},
    {"name": "THIREUS-1.7067", "bpw": 1.7067, "ppl": 4.9199, "size": 132.75, "url": "https://github.com/ikawrakow/ik_llama.cpp/discussions/477#discussioncomment-13914222"},
    {"name": "THIREUS-2.0622", "bpw": 2.0622, "ppl": 4.0622, "size": 159.84, "url": "https://github.com/ikawrakow/ik_llama.cpp/discussions/477#discussioncomment-13914222"},
    {"name": "IQ2_XSS", "bpw": 2.168, "ppl": 4.0078, "size": 168.55, "url": "https://huggingface.co/ubergarm/DeepSeek-TNG-R1T2-Chimera-GGUF/tree/main/IQ2_XSS"},
    {"name": "IQ2_KT", "bpw": 2.188, "ppl": 3.8887, "size": 170.29, "url": "https://huggingface.co/ubergarm/DeepSeek-TNG-R1T2-Chimera-GGUF/tree/main/IQ2_KT"},
    {"name": "THIREUS-2.5961", "bpw": 2.5961, "ppl": 3.6768, "size": 204.39, "url": "https://github.com/ikawrakow/ik_llama.cpp/discussions/477#discussioncomment-13883488"},
    {"name": "IQ2_KS", "bpw": 2.602, "ppl": 3.6254, "size": 204.91, "url": "https://huggingface.co/ubergarm/DeepSeek-TNG-R1T2-Chimera-GGUF/tree/main/IQ2_KS"},
    {"name": "THIREUS-2.6261", "bpw": 2.6261, "ppl": 3.5627, "size": 207.12, "url": "https://github.com/ikawrakow/ik_llama.cpp/discussions/477#discussioncomment-13914222"},
    {"name": "THIREUS-3.5753", "bpw": 3.5753, "ppl": 3.3187, "size": 280.66, "url": "https://github.com/ikawrakow/ik_llama.cpp/discussions/477#discussioncomment-13883488"},
    {"name": "THIREUS-3.5858", "bpw": 3.5858, "ppl": 3.3063, "size": 281.55, "url": "https://github.com/ikawrakow/ik_llama.cpp/discussions/477#discussioncomment-13914222"},
    {"name": "IQ3_KS", "bpw": 3.598, "ppl": 3.3167, "size": 282.60, "url": "https://huggingface.co/ubergarm/DeepSeek-TNG-R1T2-Chimera-GGUF/tree/main/IQ3_KS"}
  ]
}
{
  "title": "Kimi-K2-Instruct-0905 (1026B) Quantization Analysis",
  "subtitle": "Lower perplexity = Better performance",
  "model_parameters": 1026000000000,
  "data": [
    {"name": "smol-IQ1_KT", "bpw": 1.832, "ppl": 4.2224, "size": 227.95, "url": "https://huggingface.co/ubergarm/Kimi-K2-Instruct-0905-GGUF/tree/main/smol-IQ1_KT"},
    {"name": "smol-IQ2_KS", "bpw": 2.261, "ppl": 3.4977, "size": 281.47, "url": "https://huggingface.co/ubergarm/Kimi-K2-Instruct-0905-GGUF/tree/main/smol-IQ2_KS"},
    {"name": "IQ2_KS", "bpw": 2.425, "ppl": 3.2478, "size": 303.61, "url": "https://huggingface.co/ubergarm/Kimi-K2-Instruct-0905-GGUF/tree/main/IQ2_KS"},
    {"name": "smol-IQ2_KL", "bpw": 2.755, "ppl": 2.9294, "size": 342.81, "url": "https://huggingface.co/ubergarm/Kimi-K2-Instruct-0905-GGUF/tree/main/smol-IQ2_KL"},
    {"name": "IQ2_KL", "bpw": 3.000, "ppl": 2.7993, "size": 371.48, "url": "https://huggingface.co/ubergarm/Kimi-K2-Instruct-0905-GGUF/tree/main/IQ2_KL"},
    {"name": "smol-IQ3_KS", "bpw": 3.249, "ppl": 2.5902, "size": 401.87, "url": "https://huggingface.co/ubergarm/Kimi-K2-Instruct-0905-GGUF/tree/main/smol-IQ3_KS"},
    {"name": "IQ3_KS", "bpw": 3.520, "ppl": 2.5640, "size": 431.87, "url": "https://huggingface.co/ubergarm/Kimi-K2-Instruct-0905-GGUF/tree/main/IQ3_KS"},
    {"name": "UD-Q3_K_XL", "bpw": 3.521, "ppl": 2.6706, "size": 432.02, "url": "https://huggingface.co/unsloth/Kimi-K2-Instruct-0905-GGUF/tree/main/UD-Q3_K_XL"},
    {"name": "THIREUS-4.0285", "bpw": 4.034, "ppl": 2.493, "size": 494.61, "url": "https://github.com/ikawrakow/ik_llama.cpp/discussions/715#discussioncomment-14485602"},
    {"name": "smol-IQ4_KSS", "bpw": 4.059, "ppl": 2.5185, "size": 498.63, "url": "https://huggingface.co/ubergarm/Kimi-K2-Instruct-0905-GGUF/tree/main/smol-IQ4_KSS"},
    {"name": "IQ4_KS", "bpw": 4.633, "ppl": 2.4641, "size": 567.88, "url": "https://huggingface.co/ubergarm/Kimi-K2-Instruct-0905-GGUF/tree/main/IQ4_KS"},
    {"name": "smol-IQ5_KS", "bpw": 5.295, "ppl": 2.4526, "size": 651.80, "url": "https://huggingface.co/ubergarm/Kimi-K2-Instruct-0905-GGUF/tree/main/smol-IQ5_KS"}
  ]
}
{
  "title": "GLM-4.6 Quantization Analysis",
  "subtitle": "Lower perplexity = Better performance",
  "model_parameters": 357000000000,
  "data": [
    {"name": "smol-IQ4_KSS", "bpw": 4.090, "ppl": 3.5911, "size": 169.82, "url": "https://huggingface.co/ubergarm/GLM-4.6-GGUF/tree/main/smol-IQ4_KSS"},
    {"name": "smol-IQ1_KT", "bpw": 1.948, "ppl": 5.9034, "size": 82.31, "url": "https://huggingface.co/ubergarm/GLM-4.6-GGUF/tree/main/smol-IQ1_KT"},
    {"name": "smol-IQ2_KS", "bpw": 2.359, "ppl": 5.2760, "size": 98.97, "url": "https://huggingface.co/ubergarm/GLM-4.6-GGUF/tree/main/smol-IQ2_KS"},
    {"name": "IQ2_KL", "bpw": 3.070, "ppl": 4.1456, "size": 129.13, "url": "https://huggingface.co/ubergarm/GLM-4.6-GGUF/tree/main/IQ2_KL"},
    {"name": "IQ3_KS", "bpw": 3.573, "ppl": 3.6427, "size": 150.98, "url": "https://huggingface.co/ubergarm/GLM-4.6-GGUF/tree/main/IQ3_KS"},
    {"name": "IQ4_KS", "bpw": 4.646, "ppl": 3.5309, "size": 196.27, "url": "https://huggingface.co/ubergarm/GLM-4.6-GGUF/tree/main/IQ4_KS"},
    {"name": "IQ4_K", "bpw": 5.001, "ppl": 3.4758, "size": 210.95, "url": "https://huggingface.co/ubergarm/GLM-4.6-GGUF/tree/main/IQ4_K"},
    {"name": "THIREUS-5.5774bpw", "bpw": 5.5774, "ppl": 3.4486, "size": 234.10, "url": "https://github.com/ikawrakow/ik_llama.cpp/discussions/715#discussioncomment-14572398"},
    {"name": "UD-Q5_K_XL(unsloth)", "bpw": 5.6471, "ppl": 3.4807, "size": 238.97, "url": "https://huggingface.co/unsloth/GLM-4.6-GGUF/tree/main/UD-Q5_K_XL"},
    {"name": "IQ5_K", "bpw": 5.997, "ppl": 3.4428, "size": 254.10, "url": "https://huggingface.co/ubergarm/GLM-4.6-GGUF/tree/main/IQ5_K"},
    {"name": "Q8_0", "bpw": 8.505, "ppl": 3.4471, "size": 359.26, "url": "https://huggingface.co/ubergarm/GLM-4.6-GGUF/tree/main/Q8_0"},
    {"name": "BF16", "bpw": 16.003, "ppl": 3.4454, "size": 672.09, "url": "https://huggingface.co/ubergarm/GLM-4.6-GGUF"}
  ]
}
{
  "title": "Qwen3-Coder-480B-A35B-Instruct Quantization Analysis",
  "subtitle": "Lower perplexity = Better performance",
  "model_parameters": 480000000000,
  "data": [
    {"name": "IQ1_KT", "bpw": 1.945, "ppl": 6.3370, "size": 108.26, "url": "https://huggingface.co/ubergarm/Qwen3-Coder-480B-A35B-Instruct-GGUF/tree/main/IQ1_KT"},
    {"name": "IQ2_KS", "bpw": 2.578, "ppl": 5.6658, "size": 142.90, "url": "https://huggingface.co/ubergarm/Qwen3-Coder-480B-A35B-Instruct-GGUF/tree/main/IQ2_KS"},
    {"name": "IQ2_K", "bpw": 2.588, "ppl": 5.6578, "size": 143.49, "url": "https://huggingface.co/ubergarm/Qwen3-Coder-480B-A35B-Instruct-GGUF/tree/main/IQ2_K"},
    {"name": "IQ2_KL", "bpw": 3.034, "ppl": 5.4113, "size": 169.76, "url": "https://huggingface.co/ubergarm/Qwen3-Coder-480B-A35B-Instruct-GGUF/tree/main/IQ2_KL"},
    {"name": "IQ3_K", "bpw": 3.865, "ppl": 5.1808, "size": 214.77, "url": "https://huggingface.co/ubergarm/Qwen3-Coder-480B-A35B-Instruct-GGUF/tree/main/IQ3_K"},
    {"name": "IQ4_KSS", "bpw": 4.180, "ppl": 5.1579, "size": 236.73, "url": "https://huggingface.co/ubergarm/Qwen3-Coder-480B-A35B-Instruct-GGUF/tree/main/IQ4_KSS"},
    {"name": "IQ4_K", "bpw": 4.885, "ppl": 5.1257, "size": 276.05, "url": "https://huggingface.co/ubergarm/Qwen3-Coder-480B-A35B-Instruct-GGUF/tree/main/IQ4_K"},
    {"name": "THIREUS-5.1546bpw", "bpw": 5.1546, "ppl": 5.1057, "size": 289.53, "url": "https://github.com/ikawrakow/ik_llama.cpp/discussions/715#discussioncomment-14670424"},
    {"name": "IQ5_K", "bpw": 5.900, "ppl": 5.1073, "size": 334.84, "url": "https://huggingface.co/ubergarm/Qwen3-Coder-480B-A35B-Instruct-GGUF/tree/main/IQ5_K"},
    {"name": "Q8_0", "bpw": 8.503, "ppl": 5.0975, "size": 480.12, "url": "https://huggingface.co/ubergarm/Qwen3-Coder-480B-A35B-Instruct-GGUF/tree/main/Q8_0"}
  ]
}
```

CODE: https://github.com/ikawrakow/ik_llama.cpp/discussions/477#discussioncomment-13753552

---

## 💬 Discussion

👤 **ikawrakow** commented on **2025-08-21** at **14:34:13**

@magikRUKKOLA Thank you for these graphs, very useful! 

Can one do something to improve discoverability? I personally find it a bit hard to find which point corresponds to which quantization.

> 👤 **magikRUKKOLA** replied on **2025-08-21** at **14:59:18**
> 
> > @magikRUKKOLA Thank you for these graphs, very useful!
> > 
> > Can one do something to improve discoverability? I personally find it a bit hard to find which point corresponds to which quantization.
> 
> SVG suppose to support xlink namespace for the hyperlinks.  But I am not sure.  I have to check.

> 👤 **magikRUKKOLA** replied on **2025-08-24** at **12:04:26**
> 
> @ikawrakow 
> 
> As related to the discoverability.  I drew some opaque lines (dashed lines for non-pareto quants) so it should be easier to trace.  Not sure why the hovering effect doesn't work (in github) though.

---

👤 **ubergarm** commented on **2025-08-24** at **18:03:16**

Thanks @magikRUKKOLA  for putting these together. Always interesting to see which quantization types are performing well on some of these big models.

I just added a few more data points to my DeepSeek-V3.1 collection. The IQ4_KSS is doing unreasonably well again right around 4.0BPW.  I went back and re-read this earlier discussion on QAT and IQ4_KS here: https://github.com/ikawrakow/ik_llama.cpp/discussions/359#discussioncomment-12992769 and speculating wildly if it could have anything to do with ~4.0BPW being a "sweet spot" in the size vs perplexity trade-off curve.

<img width="2053" height="1400" alt="image" src="https://github.com/user-attachments/assets/ee3fac41-2a8f-4836-8fce-7b5997503e20" />


<details>

<summary>👈 json data</summary>

```json
[
  {
    "name": "BF16",
    "ppl": "3.3469 +/- 0.01936",
    "size": 1250.084,
    "bpw": 16.003,
    "legend": "pure"
  },
  {
    "name": "Q8_0",
    "ppl": "3.3473 +/- 0.01935",
    "size": 664.295,
    "bpw": 8.504,
    "legend": "pure",
    "skip": true
  },
  {
    "name": "IQ5_K",
    "ppl": "3.3550 +/- 0.01942",
    "size": 465.075,
    "bpw": 5.944,
    "legend": "ubergarm"
  },
  {
    "name": "IQ4_K",
    "ppl": "3.3715 +/- 0.01956",
    "size": 384.765,
    "bpw": 4.925,
    "legend": "ubergarm",
    "comment": ""
  },
  {
    "name": "IQ4_KS",
    "ppl": "3.3806 +/- 0.01966",
    "size": 363.151,
    "bpw": 4.649,
    "legend": "ubergarm",
    "comment": ""
  },
  {
    "name": "Q4_0",
    "ppl": "3.4277 +/- 0.02000",
    "size": 352.096,
    "bpw": 4.507,
    "legend": "pure",
    "comment": "q4_K embd, q6_K head"
  },
  {
    "name": "IQ4_KSS",
    "ppl": "3.3887 +/- 0.01968",
    "size": 325.088,
    "bpw": 4.162,
    "legend": "ubergarm",
    "comment": ""
  },
  {
    "name": "smol-IQ4_KSS",
    "ppl": "3.3898 +/- 0.01964",
    "size": 318.745,
    "bpw": 4.080,
    "legend": "ubergarm",
    "comment": ""
  },
  {
    "name": "IQ3_K",
    "ppl": "3.4260 +/- 0.01995",
    "size": 293.177,
    "bpw": 3.753,
    "legend": "ubergarm",
    "comment": "PR624 ik/quantization_tweaks"
  },
  {
    "name": "IQ3_KS",
    "ppl": "3.4534 +/- 0.02019",
    "size": 277.397,
    "bpw": 3.551,
    "legend": "ubergarm",
    "comment": "PR624 ik/quantization_tweaks"
  },
  {
    "name": "IQ2_KL",
    "ppl": "3.6312 +/- 0.02161",
    "size": 231.206,
    "bpw": 2.960,
    "legend": "ubergarm",
    "comment": "PR624 ik/quantization_tweaks"
  },
  {
    "name": "IQ2_KT",
    "ppl": "3.8109 +/- 0.02294",
    "size": 204.592,
    "bpw": 2.619,
    "legend": "ubergarm",
    "comment": "PR624 ik/quantization_tweaks + PR to fix KT quantization"
  },
  {
    "name": "IQ2_KS",
    "ppl": "3.9583 +/- 0.02433",
    "size": 193.144,
    "bpw": 2.472,
    "legend": "ubergarm",
    "comment": "PR624 ik/quantization_tweaks"
  },
  {
    "name": "IQ1_KT",
    "ppl": "4.3987 +/- 0.02786",
    "size": 154.968,
    "bpw": 1.984,
    "legend": "ubergarm",
    "comment": ""
  },
  {
    "name": "IQ1_S",
    "ppl": "5.3113 +/- 0.03507",
    "size": 133.610,
    "bpw": 1.710,
    "legend": "ubergarm",
    "comment": ""
  }
]
```

</details>

---

👤 **oovloveme** commented on **2025-08-29** at **01:04:16**

Add my test result :
DeepSeek-V3.1-UD-Q4_K_XL :  PPL = 3.4013 +/- 0.01980

Kimi-K2-Instruct-UD-Q3_K_XL : PPL = 3.2330 +/- 0.01668
Kimi-K2-Instruct-IQ3_KS :          PPL = 3.0275 +/- 0.01520
Kimi-K2-Instruct-IQ4_XS(Unsloth) :   PPL = 2.9812 +/- 0.01485

> 👤 **magikRUKKOLA** replied on **2025-08-30** at **04:41:43**
> 
> > Add my test result : DeepSeek-V3.1-UD-Q4_K_XL : PPL = 3.4013 +/- 0.01980
> > 
> > Kimi-K2-Instruct-UD-Q3_K_XL : PPL = 3.2330 +/- 0.01668 Kimi-K2-Instruct-IQ3_KS : PPL = 3.0275 +/- 0.01520 Kimi-K2-Instruct-IQ4_XS(Unsloth) : PPL = 2.9812 +/- 0.01485
> 
> DeepSeek-V3.1-UD-Q4_K_XL and Kimi-K2-Instruct-IQ4_XS(Unsloth) are added.  The rest already were in the graphs though with slightly different PPLs.  So the main takeaway is that we should also somehow specify the version of the ik_llama.cpp -- it seems that the way different versions can have slightly different ppls due to the changes in the code, that is, the way the quants are getting processed. see here:  https://github.com/ikawrakow/ik_llama.cpp/pull/624#issuecomment-3090335638
> 
> Also, for the future submissions plz add the BPW parameter if you can.  Its written in the logs of the llama-perplexity.  I calculated the BPW like this:
> 
> ```bash
> echo "scale=8;(452*1000^3*8)/1026000000000"|bc
> 3.52436647
> ```
> 
> where 452 is taken from the huggingface.com info page.  Hopefully the calculations are correct.

> 👤 **saood06** replied on **2025-08-30** at **11:15:53**
> 
> >Kimi-K2-Instruct-IQ4_XS(Unsloth) are added. 
> 
> You label it as IQ4_KS not IQ4_XS on the graph.

> 👤 **magikRUKKOLA** replied on **2025-08-30** at **12:10:34**
> 
> @saood06 
> > You label it as IQ4_KS not IQ4_XS on the graph.
> 
> Good catch! :D

---

👤 **magikRUKKOLA** commented on **2025-09-23** at **09:40:18**

@Thireus 

I tried to calculate the Kimi-K2-Instruct-0905-THIREUS-IQ3_K-SPECIAL_SPLIT and got very bad results.

```
system_info: n_threads = 64 / 128 | AVX = 1 | AVX_VNNI = 0 | AVX2 = 1 | AVX512 = 0 | AVX512_VBMI = 0 | AVX512_VNNI = 0 | AVX512_BF16 = 0 | FMA = 1 | NEON = 0 | SVE = 0 | ARM_FMA = 0 | F16C = 1 | FP16_VA = 0
 | WASM_SIMD = 0 | BLAS = 1 | SSE3 = 1 | SSSE3 = 1 | VSX = 0 | MATMUL_INT8 = 0 | LLAMAFILE = 1 |                                                                                 
perplexity: tokenizing the input ..                                                                                                                                              
perplexity: tokenization took 526.628 ms                                                                                                                                         
perplexity: calculating perplexity over 568 chunks, n_ctx=512, batch_size=8192, n_seq=16                                                                                         
perplexity: 60.91 seconds per pass - ETA 36.03 minutes                                                                                                                           
[1]2.3068,[2]3.1089,[3]2.4672,[4]2.1591,[5]1.9721,[6]1.8616,[7]1.8202,[8]1.7345,[9]1.7025,[10]1.6619,[11]1.6390,[12]1.6457,[13]1.6348,[14]1.6720,[15]1.7652,[16]1.8718,[17]1.9859,[18]2.1667,[19]2.1733,[20]2.
2048,[21]2.2572,[22]2.2590,[23]2.2362,[24]2.2105,[25]2.1984,[26]2.1843,[27]2.1744,[28]2.1829,[29]2.1701,[30]2.2017,[31]2.2438,[32]2.2629,[33]2.2880,[34]2.3160,[35]2.3683,[36]2.3902,[37]2.4119,[38]2.4664,[39
]2.5019,[40]2.5358,[41]2.5895,[42]2.6154,[43]2.6315,[44]2.6476,[45]2.7203,[46]2.7726,[47]2.7424,[48]2.7009,[49]2.6726,[50]2.6733,[51]2.7000,[52]2.7265,[53]2.7676,[54]2.7754,[55]2.7862,[56]2.8005,[57]2.7898,
[58]2.7912,[59]2.8104,[60]2.8433,[61]2.8745,[62]2.9012,[63]2.9306,[64]2.9514,[65]2.9715,[66]2.9618,[67]2.9505,[68]2.9331,[69]2.9371,[70]2.9336,[71]2.9101,[72]2.8962,[73]2.8847,[74]2.9014,[75]2.9051,[76]2.88
81,[77]2.8647,[78]2.8355,[79]2.8250,[80]2.8070,[81]2.7902,[82]2.7736,[83]2.7998,[84]2.7935,[85]2.7681,[86]2.7537,[87]2.7432,[88]2.7318,[89]2.7144,[90]2.7130,[91]2.6967,[92]2.6802,[93]2.6705,[94]2.6543,[95]2
.6389,[96]2.6228,[97]2.6226,[98]2.6211,[99]2.6106,[100]2.5998,[101]2.5969,[102]2.6002,[103]2.6088,[104]2.6271,[105]2.6571,[106]2.6620,[107]2.6929,[108]2.7209,[109]2.7404,[110]2.7739,[111]2.8097,[112]2.8287,
[113]2.8178,[114]2.8081,[115]2.7983,[116]2.7860,[117]2.7788,[118]2.7718,[119]2.7609,[120]2.7553,[121]2.7443,[122]2.7319,[123]2.7233,[124]2.7156,[125]2.7049,[126]2.6897,[127]2.6816,[128]2.6804,[129]2.6799,[1
30]2.6730,[131]2.6663,[132]2.6668,[133]2.6563,[134]2.6588,[135]2.6750,[136]2.6686,[137]2.6633,[138]2.6608,[139]2.6587,[140]2.6677,[141]2.6649,[142]2.6635,[143]2.6655,[144]2.6643,[145]2.6665,[146]2.6671,[147
]2.6544,[148]2.6495,[149]2.6450,[150]2.6440,[151]2.6429,[152]2.6392,[153]2.6370,[154]2.6337,[155]2.6297,[156]2.6286,[157]2.6320,[158]2.6287,[159]2.6314,[160]2.6290,[161]2.6252,[162]2.6302,[163]2.6286,[164]2
.6421,[165]2.6432,[166]2.6537,[167]2.6629,[168]2.6782,[169]2.6929,[170]2.7112,[171]2.7307,[172]2.7532,[173]2.7688,[174]2.7592,[175]2.7485,[176]2.7419,[177]2.7391,[178]2.7356,[179]2.7284,[180]2.7191,[181]2.7
139,[182]2.7170,[183]2.7319,[184]2.7468,[185]2.7616,[186]2.7761,[187]2.7865,[188]2.8020,[189]2.8181,[190]2.8327,[191]2.8426,[192]2.8456,[193]2.8535,[194]2.8573,[195]2.8559,[196]2.8632,[197]2.8669,[198]2.879
3,[199]2.8904,[200]2.8934,[201]2.9011,[202]2.8991,[203]2.9137,[204]2.9129,[205]2.9201,[206]2.9219,[207]2.9251,[208]2.9290,[209]2.9366,[210]2.9434,[211]2.9503,[212]2.9531,[213]2.9534,[214]2.9563,[215]2.9570,
[216]2.9613,[217]2.9718,[218]2.9702,[219]2.9692,[220]2.9674,[221]2.9696,[222]2.9715,[223]2.9741,[224]2.9755,[225]2.9758,[226]2.9817,[227]2.9868,[228]2.9760,[229]2.9767,[230]2.9736,[231]2.9717,[232]2.9788,[2
33]2.9869,[234]2.9921,[235]2.9835,[236]2.9785,[237]2.9784,[238]2.9821,[239]2.9862,[240]2.9908,[241]2.9976,[242]3.0043,[243]3.0117,[244]3.0190,[245]3.0307,[246]3.0387,[247]3.0412,[248]3.0468,[249]3.0523,[250
]3.0526,[251]3.0455,[252]3.0352,[253]3.0258,[254]3.0206,[255]3.0173,[256]3.0156,[257]3.0152,[258]3.0150,[259]3.0134,[260]3.0088,[261]3.0061,[262]3.0022,[263]2.9989,[264]2.9959,[265]2.9924,[266]2.9904,[267]2
.9890,[268]2.9847,[269]2.9826,[270]2.9760,[271]2.9717,[272]2.9688,[273]2.9655,[274]2.9662,[275]2.9600,[276]2.9590,[277]2.9552,[278]2.9541,[279]2.9494,[280]2.9496,[281]2.9547,[282]2.9597,[283]2.9667,[284]2.9
746,[285]2.9816,[286]2.9871,[287]2.9990,[288]3.0057,[289]3.0128,[290]3.0125,[291]3.0148,[292]3.0172,[293]3.0216,[294]3.0134,[295]3.0138,[296]3.0198,[297]3.0216,[298]3.0257,[299]3.0302,[300]3.0321,[301]3.037
2,[302]3.0432,[303]3.0423,[304]3.0408,[305]3.0431,[306]3.0428,[307]3.0460,[308]3.0450,[309]3.0460,[310]3.0471,[311]3.0474,[312]3.0418,[313]3.0376,[314]3.0354,[315]3.0330,[316]3.0300,[317]3.0262,[318]3.0226,
[319]3.0189,[320]3.0159,[321]3.0102,[322]3.0046,[323]3.0007,[324]2.9958,[325]2.9940,[326]2.9881,[327]2.9851,[328]2.9815,[329]2.9803,[330]2.9749,[331]2.9781,[332]2.9727,[333]2.9743,[334]2.9746,[335]2.9776,[3
36]2.9808,[337]2.9819,[338]2.9821,[339]2.9824,[340]2.9826,[341]2.9827,[342]2.9889,[343]2.9903,[344]2.9892,[345]2.9965,[346]3.0022,[347]3.0055,[348]3.0020,[349]2.9987,[350]2.9954,[351]2.9936,[352]2.9872,[353
]2.9821,[354]2.9776,[355]2.9727,[356]2.9679,[357]2.9651,[358]2.9616,[359]2.9585,[360]2.9568,[361]2.9536,[362]2.9494,[363]2.9460,[364]2.9417,[365]2.9373,[366]2.9323,[367]2.9272,[368]2.9228,[369]2.9188,[370]2
.9146,[371]2.9104,[372]2.9065,[373]2.9016,[374]2.8986,[375]2.8969,[376]2.8935,[377]2.8904,[378]2.8880,[379]2.8852,[380]2.8827,[381]2.8820,[382]2.8793,[383]2.8761,[384]2.8755,[385]2.8782,[386]2.8830,[387]2.8
879,[388]2.8936,[389]2.8964,[390]2.9009,[391]2.9065,[392]2.9081,[393]2.9020,[394]2.8990,[395]2.8942,[396]2.8909,[397]2.8872,[398]2.8831,[399]2.8778,[400]2.8722,[401]2.8664,[402]2.8608,[403]2.8561,[404]2.852
7,[405]2.8481,[406]2.8431,[407]2.8369,[408]2.8317,[409]2.8267,[410]2.8222,[411]2.8169,[412]2.8129,[413]2.8090,[414]2.8048,[415]2.8045,[416]2.8026,[417]2.8020,[418]2.7998,[419]2.7962,[420]2.7919,[421]2.7872,
[422]2.7869,[423]2.7830,[424]2.7819,[425]2.7789,[426]2.7765,[427]2.7729,[428]2.7687,[429]2.7674,[430]2.7633,[431]2.7588,[432]2.7545,[433]2.7504,[434]2.7484,[435]2.7461,[436]2.7419,[437]2.7375,[438]2.7335,[4
39]2.7305,[440]2.7265,[441]2.7229,[442]2.7206,[443]2.7177,[444]2.7173,[445]2.7207,[446]2.7259,[447]2.7314,[448]2.7295,[449]2.7276,[450]2.7289,[451]2.7325,[452]2.7365,[453]2.7382,[454]2.7416,[455]2.7438,[456
]2.7495,[457]2.7513,[458]2.7535,[459]2.7576,[460]2.7588,[461]2.7628,[462]2.7652,[463]2.7723,[464]2.7774,[465]2.7802,[466]2.7804,[467]2.7806,[468]2.7798,[469]2.7839,[470]2.7818,[471]2.7803,[472]2.7822,[473]2
.7830,[474]2.7824,[475]2.7842,[476]2.7843,[477]2.7841,[478]2.7859,[479]2.7854,[480]2.7863,[481]2.7857,[482]2.7848,[483]2.7841,[484]2.7833,[485]2.7831,[486]2.7854,[487]2.7837,[488]2.7864,[489]2.7856,[490]2.7
940,[491]2.7982,[492]2.8032,[493]2.8034,[494]2.8065,[495]2.8106,[496]2.8132,[497]2.8161,[498]2.8204,[499]2.8210,[500]2.8214,[501]2.8230,[502]2.8252,[503]2.8275,[504]2.8281,[505]2.8323,[506]2.8357,[507]2.842
5,[508]2.8428,[509]2.8443,[510]2.8459,[511]2.8504,[512]2.8563,[513]2.8607,[514]2.8627,[515]2.8597,[516]2.8575,[517]2.8556,[518]2.8523,[519]2.8491,[520]2.8481,[521]2.8475,[522]2.8449,[523]2.8438,[524]2.8434,
[525]2.8421,[526]2.8396,[527]2.8390,[528]2.8385,[529]2.8397,[530]2.8378,[531]2.8373,[532]2.8366,[533]2.8353,[534]2.8350,[535]2.8337,[536]2.8330,[537]2.8325,[538]2.8283,[539]2.8255,[540]2.8232,[541]2.8227,[5
42]2.8235,[543]2.8247,[544]2.8252,[545]2.8238,[546]2.8234,[547]2.8227,[548]2.8260,[549]2.8262,[550]2.8264,[551]2.8280,[552]2.8240,[553]2.8208,[554]2.8164,[555]2.8131,[556]2.8099,[557]2.8063,[558]2.8024,[559
]2.7996,[560]2.7980,[561]2.7955,[562]2.7925,[563]2.7907,[564]2.7890,[565]2.7872,[566]2.7881,[567]2.7874,[568]2.7851,                                                                                          
Final estimate: PPL = 2.7851 +/- 0.01394                                                               

llama_print_timings:        load time =    6745.76 ms                                                  
llama_print_timings:      sample time =       0.00 ms /     1 runs   (    0.00 ms per token,      inf tokens per second)                                                                                      
llama_print_timings: prompt eval time = 2136780.65 ms / 290816 tokens (    7.35 ms per token,   136.10 tokens per second)                                                                                     
llama_print_timings:        eval time =       0.00 ms /     1 runs   (    0.00 ms per token,      inf tokens per second)                                                                                      
llama_print_timings:       total time = 2142688.10 ms / 290817 tokens
```

PPL 2.7851 with 3.4325bpw?  Seems like something is very wrong.  I made sure all the split files are valild.  Anyways, honestly, its unlikely I will be using Kimi-K2 personally.  The DeepSeek-V3.1-Terminus just got released. :)

> 👤 **Thireus** replied on **2025-09-23** at **09:43:22**
> 
> Do you mean you have downloaded the huggingface repo and calculated the PPL instead of cooking a recipe?

> 👤 **magikRUKKOLA** replied on **2025-09-23** at **09:47:14**
> 
> @Thireus 
> > Do you mean you have downloaded the huggingface repo and calculated the PPL instead of cooking a recipe?
> 
> Yeah.
> 
> ```
> llama-server --version
> version: 3884 (6d2e7ca4)
> built with cc (Debian 15.2.0-4) 15.2.0 for x86_64-linux-gnu
> ```

> 👤 **Thireus** replied on **2025-09-23** at **09:54:05**
> 
> No surprise then, that's not how these repos are meant to be used. I mention it in the repo's readme.
> 
> If you download the repo, you effectively download a model where pretty much all the layers and tensors are quantised at the advertised quant type, which we know is definitely going to hurt the model PPL especially for sensitive tensors like output.weight which requires at least q6_0 and above.
> 
> You are meant to use a recipe produced with https://gguf.thireus.com and use quant_downloader.sh.
> 
> Recipe files enumerate the different quant types and optimum combination of quant types to obtain for each tensor - combination that is optimised not to hurt the PPL.
> 
> quant_downloader.sh is then fetching the individual quantised tensors from each of these huggingface repos. Mix and matching the quantisation types mentioned in the recipe file provided.

> 👤 **magikRUKKOLA** replied on **2025-09-23** at **11:00:40**
> 
> @Thireus 
> 
> Something like this:?
> 
> ```
> #!/usr/bin/env bash
> source .python/bin/activate
> cd GGUF-Tool-Suite/
> #git pull
> ln -rsf models/Kimi-K2-Instruct-0905/download.conf ./
> ln -rsf models/Kimi-K2-Instruct-0905/ppl_results.csv ./
> python3 quant_assign.py ppl_results.csv \
>  --tolerance 0.01 \
>  --cpu-irq-k 1.5 \
>  --gpu-irq-k 1.5 \
>  --gpu-assign-qtype iq6_k \
>  --cpu-tensors-max-size 490 \
>  --gpu-tensors-max-size 95% \
>  --exponential-factor 8 \
>  --cpu-tensors 'blk\..*\.ffn_down_exps\.weight' 'blk\..*\.ffn_up_exps\.weight' 'blk\..*\.ffn_gate_exps\.weight' \
>  --gpu-tensors '.*' \
>  --cpu-quants iq6_k iq5_ks_r4 iq4_ks \
>  --gpu-quants q8_0 iq5_k_r4 iq6_k \
>  --gpu-assign-tensors 'blk\..*\.attn_k_b\.weight=q8_0' \
>  --harmonize-tensors '^blk\..*\.ffn_up_exps.*,blk\..*\.ffn_gate_exps.*' \
>  --harmonization-technique 3
> ```
> 
> [EDIT]:
> ```
> ## Summary of tensor sizes per class
> # GPU Total: 10.219 GiB (94.9%) | 10.76 GiB max, if all were q8_0 | 8.88 GiB min, if all were iq5_k_r4
> # CPU Total: 502.031 GiB (64.2%) | 782.58 GiB max, if all were iq6_k | 502.03 GiB min, if all were iq4_ks
> # GPU+CPU Total: 512.250 GiB (79.5%)
> 
> ## Summary of tensor counts and bpw per qtype
> #
> # GPU-loaded quants:
> # QTYPE         Count   BPW     Assigned GiB    % Assigned      Max GiB (all)
> # +f32          365     32.0      0.62 GiB      -               -
> # +q8_0         61      8.5       0.25 GiB      -               -
> # q8_0          52      8.5       3.17 GiB      59.4%           5.33
> # +iq6_k        305     6.625     4.56 GiB      -               -
> # iq6_k         98      6.625     1.29 GiB      31.1%           4.16
> # iq5_k_r4      35      5.5       0.33 GiB      9.5%            3.45
> #
> # CPU-friendly quants:
> # QTYPE         Count   BPW     Assigned GiB    % Assigned      Max GiB (all)
> # iq6_k         0       6.625     0.00 GiB      0.0%            782.58
> # iq5_ks_r4     0       5.25      0.00 GiB      0.0%            620.16
> # iq4_ks        180     4.25    502.03 GiB      100.0%          502.03
> #
> # -Average BPW: 4.2870
> ```
> 
> [EDIT2]:
> 
> ```
> # - Command used:
> # quant_assign.py ppl_results.csv --tolerance 0.01 --cpu-irq-k 1.5 --gpu-irq-k 1.5 --gpu-assign-qtype iq6_k \
> # --cpu-tensors-max-size 470 --gpu-tensors-max-size 95% --exponential-factor 8 --cpu-tensors \
> # 'blk\..*\.ffn_down_exps\.weight' 'blk\..*\.ffn_up_exps\.weight' 'blk\..*\.ffn_gate_exps\.weight' --gpu-tensors '.*' \
> # --cpu-quants iq5_ks_r4 iq4_ks iq3_k --gpu-quants q8_0 iq5_k_r4 iq6_k --gpu-assign-tensors \
> # 'blk\..*\.attn_k_b\.weight=q8_0' --harmonize-tensors '^blk\..*\.ffn_up_exps.*,blk\..*\.ffn_gate_exps.*' \
> # --harmonization-technique 3
> 
> ```

> 👤 **Thireus** replied on **2025-09-23** at **11:07:24**
> 
> Yep.
> 
> You'll need to select additional lower quants for the CPU-friendly ones - as you can see they are all iq4_ks because it cannot reach the 490GB you have mentioned. Something like `--cpu-quants iq5_ks_r4 iq4_ks iq3_k` might be better if you want to stick to 490GB.

> 👤 **magikRUKKOLA** replied on **2025-09-23** at **11:11:08**
> 
> @Thireus 
> 
> Like this: ?
> 
> ```
> ## Summary of tensor sizes per class                                                                                        
> # GPU Total: 10.219 GiB (94.9%) | 10.76 GiB max, if all were q8_0 | 8.88 GiB min, if all were iq5_k_r4                      
> # CPU Total: 471.146 GiB (76.0%) | 620.16 GiB max, if all were iq5_ks_r4 | 406.05 GiB min, if all were iq3_k                
> # GPU+CPU Total: 481.365 GiB (85.5%)                                                                                        
>                                                                                                                             
> ## Summary of tensor counts and bpw per qtype                                                                               
> #                                                                                                                           
> # GPU-loaded quants:                                                                                                        
> # QTYPE   Count BPW Assigned GiB  % Assigned  Max GiB (all)                                                                 
> # +f32        365 32.0      0.62 GiB  -   -                                                                                 
> # +q8_0       61  8.5       0.25 GiB  -   -                                                                                 
> # q8_0        52  8.5       3.17 GiB  59.4%   5.33                                                                          
> # +iq6_k      305 6.625     4.56 GiB  -   -                                                                                 
> # iq6_k       98  6.625     1.29 GiB  31.1%   4.16                                                                          
> # iq5_k_r4    35  5.5       0.33 GiB  9.5%    3.45                                                                          
> #                                                                                                                           
> # CPU-friendly quants:                                                                                                      
> # QTYPE   Count BPW Assigned GiB  % Assigned  Max GiB (all)                                                                 
> # iq5_ks_r4   9   5.25     31.01 GiB  5.0%    620.16                                                                        
> # iq4_ks      102 4.25    284.48 GiB  56.7%   502.03                                                                        
> # iq3_k       69  3.4375  155.65 GiB  38.3%   406.05                                                                        
> #                                                                                                                           
> # -Average BPW: 4.0285 
> ```

> 👤 **Thireus** replied on **2025-09-23** at **11:14:27**
> 
> That looks really good yes.
> 
> After you test that recipe's PPL, you can try variations for it by adding some quants or tweaking the sizes you've chosen for CPU and GPU - for example adding iq6_k `--cpu-quants iq6_k iq5_ks_r4 iq4_ks iq3_k`. But I think this one you've just cooked should already provide an optimum PPL. Then you can use the _r4 variants to see if that boosts the speed for your specific hardware.

> 👤 **magikRUKKOLA** replied on **2025-09-24** at **07:37:54**
> 
> @Thireus 
> 
> okay cool
> 
> **THIREUS-4.0285bpw**
> ``` 
> Final estimate: PPL = 2.4930 +/- 0.01209
> ```

> 👤 **magikRUKKOLA** replied on **2025-09-30** at **09:14:02**
> 
> @Thireus , 
> 
> [Tried the Deepseek-V3.1-Terminus.]
> All good?  Well, its better in terms of PPL its better than [IQ5_K](https://huggingface.co/ubergarm/DeepSeek-V3.1-Terminus-GGUF#iq5_k-464062-gib-5941-bpw): 3.3969 vs 3.4000.
> 
> ```
> ## Summary of tensor sizes per class
> # GPU Total: 14.090 GiB (94.9%) | 14.85 GiB max, if all were q8_0 | 12.90 GiB min, if all were iq5_k_r4
> # CPU Total: 449.367 GiB (89.1%) | 504.33 GiB max, if all were iq6_k | 323.53 GiB min, if all were iq4_ks
> # GPU+CPU Total: 463.457 GiB (92.0%)
> 
> ## Summary of tensor counts and bpw per qtype
> #
> # GPU-loaded quants:
> # QTYPE         Count   BPW     Assigned GiB    % Assigned      Max GiB (all)
> # +f32          361     32.0      0.40 GiB      -               -
> # +q8_0         61      8.5       0.51 GiB      -               -
> # q8_0          39      8.5       2.60 GiB      47.0%           5.54
> # +iq6_k        305     6.625     8.41 GiB      -               -
> # iq6_k         102     6.625     1.61 GiB      37.2%           4.32
> # iq5_k_r4      44      5.5       0.56 GiB      15.7%           3.58
> #
> # CPU-friendly quants:
> # QTYPE         Count   BPW     Assigned GiB    % Assigned      Max GiB (all)
> # iq6_k         87      6.625   252.16 GiB      50.0%           504.33
> # iq5_ks_r4     81      5.25    186.05 GiB      46.6%           399.66
> # iq4_ks        6       4.25     11.16 GiB      3.4%            323.53
> #
> # -Average BPW: 5.9328
> #
> # -Notes:
> # - '+' means user-defined pre-assigned tensors, or tensor missing from csv data or f32 tensors
> # - Recipe produced on the 2025-09-28 20:31:51 UTC+0000 using Thireus' GGUF tools (https://gguf.thireus.com/)
> # - Script SHA-256: f385e17ea9998140203cc543e8e9d3635f6f1292999c58d837cc6d2d3d48b1e0
> # - Calibration dataset 'ppl_results.csv' SHA-256: 6e86f375f3cfc3724cc4eb748166dc1e6e4ea2e9496efdd0e1a1fbd090557d3a
> # - tensors.bf16.map SHA-256: d8bc873f82ff1b42cdec2726d649dbed5b4689f1ab599c51bbb85acdc19c743c
> # - tensors.bf16.map model name: DeepSeek-V3.1-Terminus-THIREUS-BF16-SPECIAL_TENSOR-01087-of-01087
> # - tensors.iq6_k.map SHA-256: 08c72c97c9fff653242749ff07e4f7d2df5749bda18e92b6dcc570df7ba455de
> # - tensors.iq6_k.map model name: DeepSeek-V3.1-Terminus-THIREUS-IQ6_K-SPECIAL_TENSOR-01087-of-01087
> # - tensors.iq5_ks_r4.map SHA-256: 2c82c1a399eca049cbb55036a9c6d952d9b0bf33d527cc7523cd95cfeefabc47
> # - tensors.iq5_ks_r4.map model name: DeepSeek-V3.1-Terminus-THIREUS-IQ5_KS_R4-SPECIAL_TENSOR-01087-of-01087
> # - tensors.iq4_ks.map SHA-256: f5f3aef55b13af62de6afa558be4ed72f5800ec67e7626c945d7a39c3b72611f
> # - tensors.iq4_ks.map model name: DeepSeek-V3.1-Terminus-THIREUS-IQ4_KS-SPECIAL_TENSOR-01087-of-01087
> # - tensors.q8_0.map SHA-256: 4ee954a837dcf1df29eaacbf412f2707503398839b4651b63ef7468c3f651dc5
> # - tensors.q8_0.map model name: DeepSeek-V3.1-Terminus-THIREUS-Q8_0-SPECIAL_TENSOR-01087-of-01087
> # - tensors.iq5_k_r4.map SHA-256: fb8a17d26cdad448d3b7d869bc80e541e1ea1b374d2f8c90da0ac53492b46ca9
> # - tensors.iq5_k_r4.map model name: DeepSeek-V3.1-Terminus-THIREUS-IQ5_K_R4-SPECIAL_TENSOR-01087-of-01087
> # - tensors.iq1_m_r4.map SHA-256: 6808abd977c6e077ad0368c83800bad009aea9adaebdaba0b47c023c41919ed0
> # - tensors.iq1_m_r4.map model name: DeepSeek-V3.1-Terminus-THIREUS-IQ1_M_R4-SPECIAL_TENSOR-01087-of-01087
> # - GPG signatures: DISABLED
> # - Command used:
> # quant_assign.py ppl_results.csv --tolerance 0.01 --cpu-irq-k 1.5 --gpu-irq-k 1.5 --gpu-assign-qtype iq6_k \
> # --cpu-tensors-max-size 470 --gpu-tensors-max-size 95% --exponential-factor 8 --cpu-tensors \
> # '^blk\.([3-9]|[1-5][0-9]|60)\.ffn_down_exps\.weight$' '^blk\.([3-9]|[1-5][0-9]|60)\.ffn_up_exps\.weight$' \
> # '^blk\.([3-9]|[1-5][0-9]|60)\.ffn_gate_exps\.weight$' --gpu-tensors '.*' --cpu-quants iq6_k iq5_ks_r4 iq4_ks \
> # --gpu-quants q8_0 iq5_k_r4 iq6_k --gpu-assign-tensors 'blk\..*\.attn_k_b\.weight=q8_0' --harmonize-tensors \
> # '^blk\..*\.ffn_up_exps.*,blk\..*\.ffn_gate_exps.*' --harmonization-technique 3
> ```
> 
> ```
> perplexity: 65.74 seconds per pass - ETA 38.42 minutes                                                                      
> [1]2.2950,[2]3.2890,[3]2.3841,[4]1.9958,[5]1.8024,[6]1.6619,[7]1.5663,[8]1.5003,[9]1.4529,[10]1.4110,[11]1.3959,[12]1.4329,[
> 13]1.4481,[14]1.5742,[15]1.7124,[16]1.7727,[17]1.9400,[18]2.0695,[19]2.0307,[20]2.0165,[21]2.1287,[22]2.0989,[23]2.0742,[24]
> 2.0893,[25]2.0550,[26]2.0304,[27]2.0779,[28]2.0826,[29]2.1330,[30]2.1646,[31]2.1957,[32]2.2100,[33]2.2475,[34]2.2943,[35]2.3
> 413,[36]2.3946,[37]2.4295,[38]2.4826,[39]2.5230,[40]2.5806,[41]2.6237,[42]2.6354,[43]2.6881,[44]2.7049,[45]2.7865,[46]2.8364
> ,[47]2.7925,[48]2.7465,[49]2.7247,[50]2.7426,[51]2.7894,[52]2.8043,[53]2.8553,[54]2.8729,[55]2.9037,[56]2.9354,[57]2.9528,[5
> 8]2.9936,[59]3.0049,[60]3.0546,[61]3.0926,[62]3.1435,[63]3.1744,[64]3.2194,[65]3.2262,[66]3.2118,[67]3.1903,[68]3.2224,[69]3
> .2193,[70]3.2376,[71]3.2565,[72]3.2700,[73]3.2825,[74]3.3075,[75]3.2884,[76]3.2390,[77]3.1918,[78]3.1865,[79]3.1610,[80]3.14
> 51,[81]3.1062,[82]3.1092,[83]3.0775,[84]3.0406,[85]3.0041,[86]2.9780,[87]2.9707,[88]2.9399,[89]2.9235,[90]2.8992,[91]2.8692,
> [92]2.8445,[93]2.8161,[94]2.7895,[95]2.7681,[96]2.7672,[97]2.7759,[98]2.7615,[99]2.7435,[100]2.7454,[101]2.7379,[102]2.7558,
> [103]2.7813,[104]2.7982,[105]2.7945,[106]2.8183,[107]2.8417,[108]2.8645,[109]2.8984,[110]2.9318,[111]2.9522,[112]2.9255,[113
> ]2.9132,[114]2.8896,[115]2.8740,[116]2.8620,[117]2.8380,[118]2.8167,[119]2.7944,[120]2.7757,[121]2.7586,[122]2.7398,[123]2.7
> 217,[124]2.7026,[125]2.6852,[126]2.6684,[127]2.6543,[128]2.6447,[129]2.6351,[130]2.6246,[131]2.6158,[132]2.6234,[133]2.6337,
> [134]2.6397,[135]2.6501,[136]2.6658,[137]2.6832,[138]2.6916,[139]2.7008,[140]2.7009,[141]2.7023,[142]2.7017,[143]2.7032,[144
> ]2.7010,[145]2.6922,[146]2.6897,[147]2.6936,[148]2.6926,[149]2.6941,[150]2.6880,[151]2.6854,[152]2.6823,[153]2.6782,[154]2.6
> 774,[155]2.6816,[156]2.6825,[157]2.6878,[158]2.6960,[159]2.6978,[160]2.7060,[161]2.7152,[162]2.7255,[163]2.7296,[164]2.7503,
> [165]2.7741,[166]2.7916,[167]2.8032,[168]2.8276,[169]2.8504,[170]2.8748,[171]2.8980,[172]2.8815,[173]2.8645,[174]2.8503,[175
> ]2.8382,[176]2.8256,[177]2.8144,[178]2.8018,[179]2.7883,[180]2.7927,[181]2.8062,[182]2.8217,[183]2.8365,[184]2.8501,[185]2.8
> 595,[186]2.8756,[187]2.8908,[188]2.9050,[189]2.9162,[190]2.9163,[191]2.9230,[192]2.9262,[193]2.9313,[194]2.9513,[195]2.9613,
> [196]2.9740,[197]2.9838,[198]2.9897,[199]2.9966,[200]2.9955,[201]3.0108,[202]3.0078,[203]3.0128,[204]3.0159,[205]3.0156,[206
> ]3.0183,[207]3.0262,[208]3.0372,[209]3.0463,[210]3.0470,[211]3.0421,[212]3.0423,[213]3.0502,[214]3.0519,[215]3.0571,[216]3.0
> 578,[217]3.0541,[218]3.0544,[219]3.0555,[220]3.0556,[221]3.0567,[222]3.0577,[223]3.0582,[224]3.0628,[225]3.0653,[226]3.0572,
> [227]3.0551,[228]3.0563,[229]3.0599,[230]3.0658,[231]3.0719,[232]3.0638,[233]3.0565,[234]3.0581,[235]3.0559,[236]3.0652,[237
> ]3.0736,[238]3.0832,[239]3.0935,[240]3.1033,[241]3.1142,[242]3.1294,[243]3.1426,[244]3.1524,[245]3.1653,[246]3.1762,[247]3.1
> 747,[248]3.1699,[249]3.1678,[250]3.1611,[251]3.1584,[252]3.1610,[253]3.1651,[254]3.1719,[255]3.1783,[256]3.1822,[257]3.1848,
> [258]3.1862,[259]3.1900,[260]3.1929,[261]3.1945,[262]3.1937,[263]3.1997,[264]3.2025,[265]3.2025,[266]3.2044,[267]3.2074,[268
> ]3.2117,[269]3.2149,[270]3.2141,[271]3.2127,[272]3.2061,[273]3.2063,[274]3.1991,[275]3.1881,[276]3.1764,[277]3.1785,[278]3.1
> 893,[279]3.1949,[280]3.2028,[281]3.2097,[282]3.2152,[283]3.2221,[284]3.2282,[285]3.2418,[286]3.2441,[287]3.2472,[288]3.2524,
> [289]3.2547,[290]3.2463,[291]3.2368,[292]3.2337,[293]3.2320,[294]3.2279,[295]3.2240,[296]3.2252,[297]3.2252,[298]3.2302,[299
> ]3.2358,[300]3.2383,[301]3.2415,[302]3.2429,[303]3.2444,[304]3.2437,[305]3.2557,[306]3.2628,[307]3.2732,[308]3.2619,[309]3.2
> 555,[310]3.2454,[311]3.2477,[312]3.2480,[313]3.2528,[314]3.2541,[315]3.2571,[316]3.2586,[317]3.2604,[318]3.2602,[319]3.2603,
> [320]3.2648,[321]3.2648,[322]3.2663,[323]3.2714,[324]3.2717,[325]3.2767,[326]3.2805,[327]3.2846,[328]3.2874,[329]3.2896,[330
> ]3.2962,[331]3.3009,[332]3.3057,[333]3.3045,[334]3.3047,[335]3.3052,[336]3.3056,[337]3.3065,[338]3.3065,[339]3.3090,[340]3.3
> 127,[341]3.3180,[342]3.3275,[343]3.3369,[344]3.3425,[345]3.3339,[346]3.3260,[347]3.3208,[348]3.3137,[349]3.3103,[350]3.3085,
> [351]3.3130,[352]3.3279,[353]3.3367,[354]3.3493,[355]3.3581,[356]3.3639,[357]3.3755,[358]3.3853,[359]3.3887,[360]3.3949,[361
> ]3.4041,[362]3.4127,[363]3.4183,[364]3.4249,[365]3.4315,[366]3.4415,[367]3.4498,[368]3.4565,[369]3.4641,[370]3.4729,[371]3.4
> 867,[372]3.4962,[373]3.4994,[374]3.5028,[375]3.5083,[376]3.5212,[377]3.5324,[378]3.5351,[379]3.5349,[380]3.5326,[381]3.5385,
> [382]3.5444,[383]3.5476,[384]3.5520,[385]3.5564,[386]3.5629,[387]3.5693,[388]3.5730,[389]3.5626,[390]3.5530,[391]3.5422,[392
> ]3.5363,[393]3.5265,[394]3.5178,[395]3.5089,[396]3.4987,[397]3.4899,[398]3.4806,[399]3.4700,[400]3.4612,[401]3.4508,[402]3.4
> 403,[403]3.4318,[404]3.4216,[405]3.4121,[406]3.4020,[407]3.3927,[408]3.3838,[409]3.3752,[410]3.3694,[411]3.3721,[412]3.3679,
> [413]3.3707,[414]3.3740,[415]3.3704,[416]3.3705,[417]3.3733,[418]3.3675,[419]3.3692,[420]3.3659,[421]3.3651,[422]3.3664,[423
> ]3.3650,[424]3.3689,[425]3.3685,[426]3.3693,[427]3.3690,[428]3.3723,[429]3.3740,[430]3.3772,[431]3.3783,[432]3.3775,[433]3.3
> 743,[434]3.3749,[435]3.3676,[436]3.3612,[437]3.3569,[438]3.3552,[439]3.3532,[440]3.3582,[441]3.3635,[442]3.3709,[443]3.3690,
> [444]3.3701,[445]3.3717,[446]3.3773,[447]3.3803,[448]3.3830,[449]3.3862,[450]3.3899,[451]3.3927,[452]3.3953,[453]3.3970,[454
> ]3.3959,[455]3.3981,[456]3.3981,[457]3.4008,[458]3.4057,[459]3.4064,[460]3.4060,[461]3.4027,[462]3.4062,[463]3.4139,[464]3.4
> 196,[465]3.4128,[466]3.4116,[467]3.4105,[468]3.4130,[469]3.4101,[470]3.4075,[471]3.4077,[472]3.4086,[473]3.4082,[474]3.4073,
> [475]3.4087,[476]3.4075,[477]3.4071,[478]3.4078,[479]3.4099,[480]3.4132,[481]3.4097,[482]3.4132,[483]3.4126,[484]3.4159,[485
> ]3.4221,[486]3.4252,[487]3.4288,[488]3.4342,[489]3.4366,[490]3.4412,[491]3.4476,[492]3.4520,[493]3.4520,[494]3.4533,[495]3.4
> 554,[496]3.4573,[497]3.4600,[498]3.4603,[499]3.4598,[500]3.4637,[501]3.4683,[502]3.4672,[503]3.4657,[504]3.4679,[505]3.4709,
> [506]3.4793,[507]3.4823,[508]3.4861,[509]3.4786,[510]3.4742,[511]3.4674,[512]3.4631,[513]3.4569,[514]3.4560,[515]3.4592,[516
> ]3.4545,[517]3.4549,[518]3.4548,[519]3.4552,[520]3.4600,[521]3.4590,[522]3.4573,[523]3.4636,[524]3.4619,[525]3.4600,[526]3.4
> 552,[527]3.4500,[528]3.4463,[529]3.4435,[530]3.4406,[531]3.4372,[532]3.4314,[533]3.4251,[534]3.4203,[535]3.4205,[536]3.4232,
> [537]3.4264,[538]3.4287,[539]3.4313,[540]3.4368,[541]3.4396,[542]3.4419,[543]3.4369,[544]3.4328,[545]3.4322,[546]3.4255,[547
> ]3.4191,[548]3.4122,[549]3.4050,[550]3.3991,[551]3.3926,[552]3.3863,[553]3.3803,[554]3.3781,[555]3.3763,[556]3.3790,[557]3.3
> 828,[558]3.3890,[559]3.3936,[560]3.3991,[561]3.3969,
> Final estimate: PPL = 3.3969 +/- 0.01985
> ```
> 
> I am building the new graph then?
> 
> [EDIT]:  Whoa!  And again, I was able to get the full 160k context with this custom quant.  With IQ5_K I was able to get less.

> 👤 **Thireus** replied on **2025-09-30** at **09:49:31**
> 
> @magikRUKKOLA, I think the recipe you have produced is not optimum because the cpu-friendly tensors are all sqeezed into your highest quant type (iq6_k getting 50% specifically). You should get a better ppl if you remove iq4_ks and add instead q8_0. Give it a try.

> 👤 **magikRUKKOLA** replied on **2025-09-30** at **16:25:28**
> 
> @Thireus
> 
> DeepSeek-V3.1-Terminus-6.2212bpw
> 
> ```
> Final estimate: PPL = 3.3949 +/- 0.01983
> ```
> 
> ```
> ## Summary of tensor sizes per class
> # GPU Total: 14.090 GiB (94.9%) | 14.85 GiB max, if all were q8_0 | 12.90 GiB min, if all were iq5_k_r4
> # CPU Total: 471.898 GiB (72.9%) | 647.06 GiB max, if all were q8_0 | 399.66 GiB min, if all were iq5_ks_r4
> # GPU+CPU Total: 485.989 GiB (83.9%)
> 
> ## Summary of tensor counts and bpw per qtype
> #
> # GPU-loaded quants:
> # QTYPE         Count   BPW     Assigned GiB    % Assigned      Max GiB (all)
> # +f32          361     32.0      0.40 GiB      -               -
> # +q8_0         61      8.5       0.51 GiB      -               -
> # q8_0          39      8.5       2.60 GiB      47.0%           5.54
> # +iq6_k        305     6.625     8.41 GiB      -               -
> # iq6_k         102     6.625     1.61 GiB      37.2%           4.32
> # iq5_k_r4      44      5.5       0.56 GiB      15.7%           3.58
> #
> # CPU-friendly quants:
> # QTYPE         Count   BPW     Assigned GiB    % Assigned      Max GiB (all)
> # q8_0          25      8.5      92.97 GiB      14.4%           647.06
> # iq6_k         61      6.625   176.80 GiB      35.1%           504.33
> # iq5_ks_r4     88      5.25    202.12 GiB      50.6%           399.66
> #
> # -Average BPW: 6.2212
> #
> # -Notes:
> # - '+' means user-defined pre-assigned tensors, or tensor missing from csv data or f32 tensors
> # - Recipe produced on the 2025-09-30 10:52:31 UTC+0000 using Thireus' GGUF tools (https://gguf.thireus.com/)
> # - Script SHA-256: f385e17ea9998140203cc543e8e9d3635f6f1292999c58d837cc6d2d3d48b1e0
> # - Calibration dataset 'ppl_results.csv' SHA-256: 6e86f375f3cfc3724cc4eb748166dc1e6e4ea2e9496efdd0e1a1fbd090557d3a
> # - tensors.bf16.map SHA-256: d8bc873f82ff1b42cdec2726d649dbed5b4689f1ab599c51bbb85acdc19c743c
> # - tensors.bf16.map model name: DeepSeek-V3.1-Terminus-THIREUS-BF16-SPECIAL_TENSOR-01087-of-01087
> # - tensors.q8_0.map SHA-256: 4ee954a837dcf1df29eaacbf412f2707503398839b4651b63ef7468c3f651dc5
> # - tensors.q8_0.map model name: DeepSeek-V3.1-Terminus-THIREUS-Q8_0-SPECIAL_TENSOR-01087-of-01087
> # - tensors.iq6_k.map SHA-256: 08c72c97c9fff653242749ff07e4f7d2df5749bda18e92b6dcc570df7ba455de
> # - tensors.iq6_k.map model name: DeepSeek-V3.1-Terminus-THIREUS-IQ6_K-SPECIAL_TENSOR-01087-of-01087
> # - tensors.iq5_ks_r4.map SHA-256: 2c82c1a399eca049cbb55036a9c6d952d9b0bf33d527cc7523cd95cfeefabc47
> # - tensors.iq5_ks_r4.map model name: DeepSeek-V3.1-Terminus-THIREUS-IQ5_KS_R4-SPECIAL_TENSOR-01087-of-01087
> # - tensors.iq5_k_r4.map SHA-256: fb8a17d26cdad448d3b7d869bc80e541e1ea1b374d2f8c90da0ac53492b46ca9
> # - tensors.iq5_k_r4.map model name: DeepSeek-V3.1-Terminus-THIREUS-IQ5_K_R4-SPECIAL_TENSOR-01087-of-01087
> # - tensors.iq1_m_r4.map SHA-256: 6808abd977c6e077ad0368c83800bad009aea9adaebdaba0b47c023c41919ed0
> # - tensors.iq1_m_r4.map model name: DeepSeek-V3.1-Terminus-THIREUS-IQ1_M_R4-SPECIAL_TENSOR-01087-of-01087
> # - GPG signatures: DISABLED
> # - Command used:
> # quant_assign.py ppl_results.csv --tolerance 0.01 --cpu-irq-k 1.5 --gpu-irq-k 1.5 --gpu-assign-qtype iq6_k \
> # --cpu-tensors-max-size 470 --gpu-tensors-max-size 95% --exponential-factor 8 --cpu-tensors \
> # '^blk\.([3-9]|[1-5][0-9]|60)\.ffn_down_exps\.weight$' '^blk\.([3-9]|[1-5][0-9]|60)\.ffn_up_exps\.weight$' \
> # '^blk\.([3-9]|[1-5][0-9]|60)\.ffn_gate_exps\.weight$' --gpu-tensors '.*' --cpu-quants q8_0 iq6_k iq5_ks_r4 \
> # --gpu-quants q8_0 iq5_k_r4 iq6_k --gpu-assign-tensors 'blk\..*\.attn_k_b\.weight=q8_0' --harmonize-tensors \
> # '^blk\..*\.ffn_up_exps.*,blk\..*\.ffn_gate_exps.*' --harmonization-technique 3
> ```

> 👤 **Thireus** replied on **2025-09-30** at **16:30:19**
> 
> Interesting, so really minimal improvement. In that case you may try to add bf16 to the list of GPU-friendly tensors. I have discovered that it improves the ppl for some models but not sure about DeepSeek.

> 👤 **magikRUKKOLA** replied on **2025-10-01** at **03:24:42**
> 
> @Thireus
> 
> ```
> Final estimate: PPL = 3.3975 +/- 0.01985
> ```
> 
> ```
> ## Summary of tensor sizes per class
> # GPU Total: 18.296 GiB (92.7%) | 19.74 GiB max, if all were bf16 | 12.90 GiB min, if all were iq5_k_r4
> # CPU Total: 471.898 GiB (72.9%) | 647.06 GiB max, if all were q8_0 | 399.66 GiB min, if all were iq5_ks_r4
> # GPU+CPU Total: 490.195 GiB (82.8%)
> 
> ## Summary of tensor counts and bpw per qtype
> #
> # GPU-loaded quants:
> # QTYPE         Count   BPW     Assigned GiB    % Assigned      Max GiB (all)
> # +f32          361     32.0      0.40 GiB      -               -
> # bf16          92      16.0      7.44 GiB      71.4%           10.42
> # +q8_0         61      8.5       0.51 GiB      -               -
> # q8_0          84      8.5       1.45 GiB      26.2%           5.54
> # +iq6_k        305     6.625     8.41 GiB      -               -
> # iq6_k         0       6.625     0.00 GiB      0.0%            4.32
> # iq5_k_r4      9       5.5       0.08 GiB      2.4%            3.58
> #
> # CPU-friendly quants:
> # QTYPE         Count   BPW     Assigned GiB    % Assigned      Max GiB (all)
> # q8_0          25      8.5      92.97 GiB      14.4%           647.06
> # iq6_k         61      6.625   176.80 GiB      35.1%           504.33
> # iq5_ks_r4     88      5.25    202.12 GiB      50.6%           399.66
> #
> # -Average BPW: 6.2751
> #
> # -Notes:
> # - '+' means user-defined pre-assigned tensors, or tensor missing from csv data or f32 tensors
> # - Recipe produced on the 2025-09-30 19:55:51 UTC+0000 using Thireus' GGUF tools (https://gguf.thireus.com/)
> # - Script SHA-256: f385e17ea9998140203cc543e8e9d3635f6f1292999c58d837cc6d2d3d48b1e0
> # - Calibration dataset 'ppl_results.csv' SHA-256: 6e86f375f3cfc3724cc4eb748166dc1e6e4ea2e9496efdd0e1a1fbd090557d3a
> # - tensors.bf16.map SHA-256: d8bc873f82ff1b42cdec2726d649dbed5b4689f1ab599c51bbb85acdc19c743c
> # - tensors.bf16.map model name: DeepSeek-V3.1-Terminus-THIREUS-BF16-SPECIAL_TENSOR-01087-of-01087
> # - tensors.q8_0.map SHA-256: 4ee954a837dcf1df29eaacbf412f2707503398839b4651b63ef7468c3f651dc5
> # - tensors.q8_0.map model name: DeepSeek-V3.1-Terminus-THIREUS-Q8_0-SPECIAL_TENSOR-01087-of-01087
> # - tensors.iq6_k.map SHA-256: 08c72c97c9fff653242749ff07e4f7d2df5749bda18e92b6dcc570df7ba455de
> # - tensors.iq6_k.map model name: DeepSeek-V3.1-Terminus-THIREUS-IQ6_K-SPECIAL_TENSOR-01087-of-01087
> # - tensors.iq5_ks_r4.map SHA-256: 2c82c1a399eca049cbb55036a9c6d952d9b0bf33d527cc7523cd95cfeefabc47
> # - tensors.iq5_ks_r4.map model name: DeepSeek-V3.1-Terminus-THIREUS-IQ5_KS_R4-SPECIAL_TENSOR-01087-of-01087
> # - tensors.iq5_k_r4.map SHA-256: fb8a17d26cdad448d3b7d869bc80e541e1ea1b374d2f8c90da0ac53492b46ca9
> # - tensors.iq5_k_r4.map model name: DeepSeek-V3.1-Terminus-THIREUS-IQ5_K_R4-SPECIAL_TENSOR-01087-of-01087
> # - tensors.iq1_m_r4.map SHA-256: 6808abd977c6e077ad0368c83800bad009aea9adaebdaba0b47c023c41919ed0
> # - tensors.iq1_m_r4.map model name: DeepSeek-V3.1-Terminus-THIREUS-IQ1_M_R4-SPECIAL_TENSOR-01087-of-01087
> # - GPG signatures: DISABLED
> # - Command used:
> # quant_assign.py ppl_results.csv --tolerance 0.01 --cpu-irq-k 1.5 --gpu-irq-k 1.5 --gpu-assign-qtype iq6_k \
> # --cpu-tensors-max-size 470 --gpu-tensors-max-size 95% --exponential-factor 8 --cpu-tensors \
> # '^blk\.([3-9]|[1-5][0-9]|60)\.ffn_down_exps\.weight$' '^blk\.([3-9]|[1-5][0-9]|60)\.ffn_up_exps\.weight$' \
> # '^blk\.([3-9]|[1-5][0-9]|60)\.ffn_gate_exps\.weight$' --gpu-tensors '.*' --cpu-quants q8_0 iq6_k iq5_ks_r4 \
> # --gpu-quants bf16 q8_0 iq5_k_r4 iq6_k --gpu-assign-tensors 'blk\..*\.attn_k_b\.weight=q8_0' --harmonize-tensors \
> # '^blk\..*\.ffn_up_exps.*,blk\..*\.ffn_gate_exps.*' --harmonization-technique 3
> 
> ```

> 👤 **Thireus** replied on **2025-10-01** at **06:23:09**
> 
> @magikRUKKOLA, interesting, so we're hitting a hard ceiling here. Are you using `-ctk f16 -c 512 -b 4096 -ub 4096` to compute the PPL?
> 
> Pure Q8_0 is reported to be 3.3929, so we're pretty much there already. And 5.339bpw (smol-IQ5_KS) is reported at 3.4059. So, indeed any bpw above 5 should be below 3.41 but I assume above 3.39 so we won't see much gain. I don't have BF16 ppl, but I bet it's above 3.39.

> 👤 **magikRUKKOLA** replied on **2025-10-01** at **13:17:12**
> 
> @Thireus 
> 
> > Are you using -ctk f16 -c 512 -b 4096 -ub 4096 to compute the PPL?
> 
> Not exactly that.  I am using the following:
> 
> ```
>     --ctx-size $((512)) \                                                                                                   
>     -b $((16 * 512)) -ub $((8 * 512)) \                                                                                     
>     -ctk q8_0 \                               
> ```
> 
> Should I retest with -ctk f16?

> 👤 **Thireus** replied on **2025-10-01** at **13:51:15**
> 
> I have a feeling these would produce a different ppl yes.

> 👤 **magikRUKKOLA** replied on **2025-10-01** at **15:23:16**
> 
> @Thireus 
> > I have a feeling these would produce a different ppl yes.
> 
> DeepSeek-V3.1-Terminus-6.2751bpw:
> ```
> Final estimate: PPL = 3.3960 +/- 0.01984
> ```
> 
> So the best quant that fits into 512GB RAM is DeepSeek-V3.1-Terminus-6.2212bpw then?

> 👤 **Thireus** replied on **2025-10-01** at **16:08:17**
> 
> I would say that with such small variations of ppl, any one that gives you 3.39xx ppl is very much like running the unquantised version of the model. So, I'd go for the one that fits all the context you need for your use-case.
> 
> For my personal use, I always try to fit everything into VRAM only since it gives the fastest speed. Let's say I have 300GB of VRAM, I compute a first recipe with a max size of 200GB in total, and then I check how much context I can fit in the remaining free VRAM with `-ctk q8_0`. I then compute the ppl with `-ctk f16`, and if there's still some VRAM available I try to produce a new recipe with increased size. If I discover that the ppl doesn't improve by much, let's say we're already past [the elbow curve after 5bpw](https://github.com/Thireus/GGUF-Tool-Suite/blob/main/ppl_graphs/DeepSeek-V3.1-Terminus.svg) then I just happily stop. I may sometimes check if using different quants would improve my speeds, for example we know _kt quants are slower but provide better ppl for their sizes, so if I still have plenty of VRAM available I would switch away from _kt quants and use the normal _k quants knowing that for the same ppl target I would actually consume more VRAM (and I'll be ok with that).

> 👤 **magikRUKKOLA** replied on **2025-10-01** at **17:32:53**
> 
> @Thireus 
> > I would say that with such small variations of ppl, any one that gives you 3.39xx ppl is very much like running the unquantised version of the model. So, I'd go for the one that fits all the context you need for your use-case.
> > 
> 
> Cool.  Then I will use the 5.9328bpw.  It fits my 72GB VRAM with 160k ctx.
> Or, it I should search for lower bpw quant which gives 3.39xx ppl to improve the speed?

> 👤 **Thireus** replied on **2025-10-01** at **17:36:17**
> 
> I think that one was good. I don't think you can improve much further, unless you are using AMD and want to try the _r4 quant variants when available for better speed.

> 👤 **magikRUKKOLA** replied on **2025-10-01** at **17:56:36**
> 
> @Thireus 
> > unless you are using AMD
> 
> Yeah, I am using Threadripper PRO 3995wx.  Okay, will try _r4 then.

> 👤 **magikRUKKOLA** replied on **2025-10-01** at **23:40:30**
> 
> @Thireus 
> > For my personal use, I always try to fit everything into VRAM only since it gives the fastest speed.
> 
> What VRAM BW are you getting with such a setup?
> For example, if I would build say, 192 GB VRAM total setup with 8 RTX 3090 and run say, GLM-4.6 quant limited to say, 168GB so the rest 24 GB will be used for a small context, what speed improvement are we talking about (if compared to CPU+GPU inference)?

> 👤 **magikRUKKOLA** replied on **2025-10-02** at **01:01:49**
> 
> @Thireus 
> 
> As related to _R4 quants.  I am not sure if they improve much for me.
> 
> I just tried to compare the IQ5_K quant from @ubergarm with and without --run-time-repack.
> 
> ```
> #!/usr/bin/env bash
> 
> custom="
> ## Attention [0-60] (GPU)
> # attn_kv_b is only used for PP so keep it q8_0 for best speed and accuracy
> blk\..*\.attn_kv_b\.weight=q8_0
> 
> # ideally k_b and v_b are smaller than q8_0 as they are is used for TG with -mla 3
> # https://github.com/ikawrakow/ik_llama.cpp/issues/651
> # blk.*.attn_k_b.weight is not divisible by 256 so only supports iq4_nl or legacy qN_0
> blk\..*\.attn_k_b\.weight=q8_0
> blk\..*\.attn_v_b\.weight=q8_0
> 
> # Balance of attn tensors
> blk\..*\.attn_kv_a_mqa\.weight=q8_0
> blk\..*\.attn_q_a\.weight=q8_0
> blk\..*\.attn_q_b\.weight=q8_0
> blk\..*\.attn_output\.weight=q8_0
> 
> ## First Three Dense Layers [0-2] (GPU)
> blk\..*\.ffn_down\.weight=q8_0
> blk\..*\.ffn_(gate|up)\.weight=q8_0
> 
> ## Shared Expert (1-60) (GPU)
> blk\..*\.ffn_down_shexp\.weight=q8_0
> blk\..*\.ffn_(gate|up)_shexp\.weight=q8_0
> 
> ## Routed Experts (1-60) (CPU)
> blk\..*\.ffn_down_exps\.weight=iq6_k
> blk\..*\.ffn_(gate|up)_exps\.weight=iq5_k
> 
> ## Token embedding and output tensors (GPU)
> token_embd\.weight=iq6_k
> output\.weight=iq6_k
> "
> ```
> 
> With -rtr the RAM bandwith during decode phase is about 5-7GB/s lower and it works slower in decode.
> 
> without -rtr:
> ```
> |    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
> |-------|--------|--------|----------|----------|----------|----------|
> |  4096 |   1024 |      0 |   33.024 |   124.03 |  141.090 |     7.26 |
> |  4096 |   1024 |   4096 |   33.627 |   121.81 |  144.304 |     7.10 |
> |  4096 |   1024 |   8192 |   34.112 |   120.08 |  149.796 |     6.84 |
> ```
> 
> with -rtr:
> ```
> main: n_kv_max = 143360, n_batch = 8192, n_ubatch = 4096, flash_attn = 1, n_gpu_layers = 99, n_threads = 64, n_threads_batch = 64
> 
> |    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
> |-------|--------|--------|----------|----------|----------|----------|
> |  4096 |   1024 |      0 |   32.392 |   126.45 |  150.104 |     6.82 |
> |  4096 |   1024 |   4096 |   32.874 |   124.60 |  154.243 |     6.64 |
> |  4096 |   1024 |   8192 |   33.418 |   122.57 |  157.535 |     6.50 |
> ```
> 
> So what that might mean?  I should NOT use R4 quants for CPU if I want higher decode speed?  I am confused.

> 👤 **Thireus** replied on **2025-10-02** at **06:59:15**
> 
> CPU+GPU (pp 127.3 t/s + tg 9.9 t/s) - https://huggingface.co/ubergarm/Kimi-K2-Instruct-0905-GGUF/discussions/1#68beb4a4441f681751abc87c
> GPU only (pp 751.6 t/s + tg 30.4 t/s) - https://huggingface.co/ubergarm/Kimi-K2-Instruct-0905-GGUF/discussions/1#68cd97a9217b07616817c6a9
> 
> Hardware: Intel i9-7980XE + 256GB DDR4 + 3x RTX 6000 Pro + 1x 5090
> 
> _r4 are only available for some quants, for example iq6_k won't benefit from it, see list here: https://github.com/Thireus/GGUF-Tool-Suite/?tab=readme-ov-file#%EF%B8%8F-generate-a-custom-recipe-for-your-config
> So if you only have a handful of CPU tensors using _r4 quants I suppose the perf gains will be minimal. For the recipe you mention, I only see "blk\..*\.ffn_(gate|up)_exps\.weight=iq5_k" CPU tensor that would use iq5_k_r4 when setting -rtr.
> 
> If no perf gains observed for pp or tg speed, then don't use -rtr.

> 👤 **magikRUKKOLA** replied on **2025-10-02** at **09:48:45**
> 
> @Thireus 
> 
> Should I checkout the following then?
> 
> ```
> ## Summary of tensor sizes per class
> # GPU Total: 14.090 GiB (94.9%) | 14.85 GiB max, if all were q8_0 | 12.90 GiB min, if all were iq5_k_r4
> # CPU Total: 406.547 GiB (97.1%) | 418.69 GiB max, if all were iq5_k_r4 | 323.53 GiB min, if all were iq4_ks_r4
> # GPU+CPU Total: 420.637 GiB (96.0%)
> 
> ## Summary of tensor counts and bpw per qtype
> #
> # GPU-loaded quants:
> # QTYPE         Count   BPW     Assigned GiB    % Assigned      Max GiB (all)
> # +f32          361     32.0      0.40 GiB      -               -
> # +q8_0         61      8.5       0.51 GiB      -               -
> # q8_0          39      8.5       2.60 GiB      47.0%           5.54
> # +iq6_k        305     6.625     8.41 GiB      -               -
> # iq6_k         102     6.625     1.61 GiB      37.2%           4.32
> # iq5_k_r4      44      5.5       0.56 GiB      15.7%           3.58
> #
> # CPU-friendly quants:
> # QTYPE         Count   BPW     Assigned GiB    % Assigned      Max GiB (all)
> # iq5_k_r4      87      5.5     209.34 GiB      50.0%           418.69
> # iq5_ks_r4     81      5.25    186.05 GiB      46.6%           399.66
> # iq4_ks_r4     6       4.25     11.16 GiB      3.4%            323.53
> #
> # -Average BPW: 5.3847
> #
> # -Notes:
> # - '+' means user-defined pre-assigned tensors, or tensor missing from csv data or f32 tensors
> # - Recipe produced on the 2025-10-01 22:26:05 UTC+0000 using Thireus' GGUF tools (https://gguf.thireus.com/)
> # - Script SHA-256: f385e17ea9998140203cc543e8e9d3635f6f1292999c58d837cc6d2d3d48b1e0
> # - Calibration dataset 'ppl_results.csv' SHA-256: 6e86f375f3cfc3724cc4eb748166dc1e6e4ea2e9496efdd0e1a1fbd090557d3a
> # - tensors.bf16.map SHA-256: d8bc873f82ff1b42cdec2726d649dbed5b4689f1ab599c51bbb85acdc19c743c
> # - tensors.bf16.map model name: DeepSeek-V3.1-Terminus-THIREUS-BF16-SPECIAL_TENSOR-01087-of-01087
> # - tensors.iq5_k_r4.map SHA-256: fb8a17d26cdad448d3b7d869bc80e541e1ea1b374d2f8c90da0ac53492b46ca9
> # - tensors.iq5_k_r4.map model name: DeepSeek-V3.1-Terminus-THIREUS-IQ5_K_R4-SPECIAL_TENSOR-01087-of-01087
> # - tensors.iq5_ks_r4.map SHA-256: 2c82c1a399eca049cbb55036a9c6d952d9b0bf33d527cc7523cd95cfeefabc47
> # - tensors.iq5_ks_r4.map model name: DeepSeek-V3.1-Terminus-THIREUS-IQ5_KS_R4-SPECIAL_TENSOR-01087-of-01087
> # - tensors.iq4_ks_r4.map SHA-256: 3b051631543d04f711aa61714cd6cabc632b1fb34ca4972dffa56e3569466d70
> # - tensors.iq4_ks_r4.map model name: DeepSeek-V3.1-Terminus-THIREUS-IQ4_KS_R4-SPECIAL_TENSOR-01087-of-01087
> # - tensors.q8_0.map SHA-256: 4ee954a837dcf1df29eaacbf412f2707503398839b4651b63ef7468c3f651dc5
> # - tensors.q8_0.map model name: DeepSeek-V3.1-Terminus-THIREUS-Q8_0-SPECIAL_TENSOR-01087-of-01087
> # - tensors.iq6_k.map SHA-256: 08c72c97c9fff653242749ff07e4f7d2df5749bda18e92b6dcc570df7ba455de
> # - tensors.iq6_k.map model name: DeepSeek-V3.1-Terminus-THIREUS-IQ6_K-SPECIAL_TENSOR-01087-of-01087
> # - tensors.iq1_m_r4.map SHA-256: 6808abd977c6e077ad0368c83800bad009aea9adaebdaba0b47c023c41919ed0
> # - tensors.iq1_m_r4.map model name: DeepSeek-V3.1-Terminus-THIREUS-IQ1_M_R4-SPECIAL_TENSOR-01087-of-01087
> # - GPG signatures: DISABLED
> # - Command used:
> # quant_assign.py ppl_results.csv --tolerance 0.01 --cpu-irq-k 1.5 --gpu-irq-k 1.5 --gpu-assign-qtype iq6_k \
> # --cpu-tensors-max-size 410 --gpu-tensors-max-size 95% --exponential-factor 8 --cpu-tensors \
> # '^blk\.([3-9]|[1-5][0-9]|60)\.ffn_down_exps\.weight$' '^blk\.([3-9]|[1-5][0-9]|60)\.ffn_up_exps\.weight$' \
> # '^blk\.([3-9]|[1-5][0-9]|60)\.ffn_gate_exps\.weight$' --gpu-tensors '.*' --cpu-quants iq5_k_r4 iq5_ks_r4 iq4_ks_r4 \
> # --gpu-quants q8_0 iq5_k_r4 iq6_k --gpu-assign-tensors 'blk\..*\.attn_k_b\.weight=q8_0' --harmonize-tensors \
> # '^blk\..*\.ffn_up_exps.*,blk\..*\.ffn_gate_exps.*' --harmonization-technique 3
> ```

> 👤 **Thireus** replied on **2025-10-02** at **10:13:38**
> 
> So, what I found often works best (as in giving the best PPL) is to only select one quant type per bit. Here you've selected iq5_k and iq5_ks which are very similar in terms of BPW, so the recipe produced is very similar to a pure iq5_ks, which defeats the "dynamic quant" purpose.
> 
> Also, you usually want to avoid creating recipes where the distribution is too heavy on the highest quant type, here I see iq4 only gets 3% which is very small compared to the iq5 ones.
> 
> For a better distribution I would say try the following, for `--cpu-tensors-max-size 410`:
> 
> - `--cpu-quants iq6_k iq5_ks_r4 iq4_ks_r4`
> - `--cpu-quants q8_0 iq6_k iq5_ks_r4 iq4_ks_r4`
> - `--cpu-quants q8_0 iq6_k iq5_ks_r4`
> - `--cpu-quants iq6_k iq5_ks_r4`
> 
> Adding iq6_k and q8_0 should allow for a better "bell" looking like distribution.
> 
> You can also refer to https://github.com/Thireus/GGUF-Tool-Suite/blob/main/quants_graphs/Best_AMD_Quants.png, it's possible that `iq5_ks_r4` is in fact slower than `iq5_ks` for your system as it was in my benchmark with the 5950X CPU. Same for `iq4_ks_r4`... If which case you should drop the _r4.

> 👤 **magikRUKKOLA** replied on **2025-10-03** at **02:09:16**
> 
> @Thireus 
> 
> Indeed, having a better prefill (~ +10%) with R4 quants but slightly worse decode (~1-2%).
> 
> THIREUS/DeepSeek-V3.1-Terminus-5.4498bpw:
> 
> > --cpu-quants iq6_k iq5_ks_r4 iq4_ks_r4
> 
> ```
> ## Summary of tensor sizes per class
> # GPU Total: 14.090 GiB (94.9%) | 14.85 GiB max, if all were q8_0 | 12.90 GiB min, if all were iq5_k_r4
> # CPU Total: 411.633 GiB (81.6%) | 504.33 GiB max, if all were iq6_k | 323.53 GiB min, if all were iq4_ks_r4
> # GPU+CPU Total: 425.723 GiB (88.2%)
> 
> ## Summary of tensor counts and bpw per qtype
> #
> # GPU-loaded quants:
> # QTYPE         Count   BPW     Assigned GiB    % Assigned      Max GiB (all)
> # +f32          361     32.0      0.40 GiB      -               -
> # +q8_0         61      8.5       0.51 GiB      -               -
> # q8_0          39      8.5       2.60 GiB      47.0%           5.54
> # +iq6_k        305     6.625     8.41 GiB      -               -
> # iq6_k         102     6.625     1.61 GiB      37.2%           4.32
> # iq5_k_r4      44      5.5       0.56 GiB      15.7%           3.58
> #
> # CPU-friendly quants:
> # QTYPE         Count   BPW     Assigned GiB    % Assigned      Max GiB (all)
> # iq6_k         49      6.625   142.02 GiB      28.2%           504.33
> # iq5_ks_r4     85      5.25    195.23 GiB      48.9%           399.66
> # iq4_ks_r4     40      4.25     74.38 GiB      23.0%           323.53
> #
> # -Average BPW: 5.4498
> ```
> 
> ```
> main: n_kv_max = 163840, n_batch = 8192, n_ubatch = 4096, flash_attn = 1, n_gpu_layers = 99, n_threads = 64, n_threads_batch = 64
> 
> |    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
> |-------|--------|--------|----------|----------|----------|----------|
> |  4096 |   1024 |      0 |   29.734 |   137.76 |  140.578 |     7.28 |
> |  4096 |   1024 |   4096 |   30.217 |   135.55 |  143.079 |     7.16 |
> |  4096 |   1024 |   8192 |   30.771 |   133.11 |  147.195 |     6.96 |
> ```
> 
> ubergarm/DeepSeek-V3.1-Terminus/IQ5_K (*n_kv_max=140k):
> ```
> |    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
> |-------|--------|--------|----------|----------|----------|----------|
> |  4096 |   1024 |      0 |   32.634 |   125.51 |  138.274 |     7.41 |
> |  4096 |   1024 |   4096 |   33.103 |   123.74 |  141.111 |     7.26 |
> |  4096 |   1024 |   8192 |   33.609 |   121.87 |  145.452 |     7.04 |
> ```
> 
> Let me see what's up with PPL.
> 
> [EDIT]:
> 
> ```
> Final estimate: PPL = 3.3961 +/- 0.01984
> ```
> 
> Ha!
> 
> [EDIT2]:
> 
> Okay cool!  Thanks alot, Bro!  A very cool framework you got indeed.
> 
> [EDIT3]:
> I made a recalculation of the ubergarm IQ5_K quant of Deepseek V3.1-Terminus (because I am using bigger batches for a speed) and its:
> ```
> Final estimate: PPL = 3.4028 +/- 0.01993
> ```
> while the declared PPL is actually:
> ```
> Final estimate: PPL = 3.4000 +/- 0.01992
> ```
> 
> So my calculations could be off by at least 0.0028 PPL.
> 
> [EDIT4]:
> 
> The performance with 8k batches and 128k ctx:
> 
> ```
> Oct 08 13:20:50 xxx run-ik_llama.cpp.sh[86360]: main: n_kv_max = 131072, n_batch = 8192, n_ubatch = 8192, flash_attn = 1, n_gpu_layers = 99, n_threads = 64, n_threads_batch = 64
> Oct 08 13:20:50 xxx run-ik_llama.cpp.sh[86360]: |    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
> Oct 08 13:20:50 xxx run-ik_llama.cpp.sh[86360]: |-------|--------|--------|----------|----------|----------|----------|
> Oct 08 13:26:31 xxx run-ik_llama.cpp.sh[86360]: |  8192 |   2048 |      0 |   35.478 |   230.90 |  266.567 |     7.68 |
> Oct 08 13:31:51 xxx run-ik_llama.cpp.sh[86360]: |  8192 |   2048 |   8192 |   39.577 |   206.99 |  279.792 |     7.32 |
> Oct 08 13:37:33 xxx run-ik_llama.cpp.sh[86360]: |  8192 |   2048 |  16384 |   48.008 |   170.64 |  294.366 |     6.96 |
> 
> ```
> 
> System usage:
> 
> ```
> ┌─NVIDIA GeForce RTX 3090─────────────────────────────────────────────────────────────────────────────┐
> │ Temp: 76C  Fan: 100%  Power: 208W                                                                   │
> │ GPU: 1995MHz (71%)  VRAM: 10251MHz (02%)  Mem: 23.1GB                                               │
> │ Temp [74-84]                     Fan [99-100]                     Power [178-394]                   │
> ││       █                        │█████   ███████████████████████ │      █                           │
> ││███████████████████████████████ │███████████████████████████████ │███████████████████████████████   │
> ││─────────────────────────────── │─────────────────────────────── │───────────────────────────────   │
> │ GPU Clock [1680-2010]            VRAM Clock [10251-10251]         Util [0-100]                      │
> ││█████                           │                                │     ██████                       │
> ││███████████████████████████████ │███████████████████████████████ │███████████████████████████████   │
> └│─────────────────────────────── │─────────────────────────────── │─────────────────────────────── ──┘
> ┌─NVIDIA GeForce RTX 3090─────────────────────────────────────────────────────────────────────────────┐
> │ Temp: 70C  Fan: 100%  Power: 310W                                                                   │
> │ GPU: 1740MHz (100%)  VRAM: 10251MHz (62%)  Mem: 22.7GB                                              │
> │ Temp [58-70]                     Fan [99-100]                     Power [147-348]                   │
> ││                           █  █ │████████████  █████████████████ │                    █             │
> ││███████████████████████████████ │███████████████████████████████ │███████████████████████████████   │
> ││─────────────────────────────── │─────────────────────────────── │───────────────────────────────   │
> │ GPU Clock [1725-2040]            VRAM Clock [10251-10251]         Util [0-100]                      │
> ││███████████  █                  │                                │             █  █ ██ █ █  ██  █   │
> ││███████████████████████████████ │███████████████████████████████ │███████████████████████████████   │
> └│─────────────────────────────── │─────────────────────────────── │─────────────────────────────── ──┘
> ┌─NVIDIA GeForce RTX 3090─────────────────────────────────────────────────────────────────────────────┐
> │ Temp: 70C  Fan: 100%  Power: 167W                                                                   │
> │ GPU: 2010MHz (00%)  VRAM: 10251MHz (00%)  Mem: 23.0GB                                               │
> │ Temp [70-73]                     Fan [0-100]                      Power [167-206]                   │
> ││███                             │███████████████████████████████ │██                                │
> ││███████████████████████████████ │███████████████████████████████ │███████████████████████████████   │
> ││─────────────────────────────── │─────────────────────────────── │───────────────────────────────   │
> │ GPU Clock [2010-2010]            VRAM Clock [10251-10251]         Util [0-19]                       │
> ││                                │                                │   ██                             │
> ││███████████████████████████████ │███████████████████████████████ │███████████████████████████████   │
> └│─────────────────────────────── │─────────────────────────────── │─────────────────────────────── ──┘
> RAM: 34.15 GB/s [0.38-100.71]                                                                          
>                                        █                                                               
> ████████████████████████████████████████████████████████████████████████████                           
> ████████████████████████████████████████████████████████████████████████████                           
> ████████████████████████████████████████████████████████████████████████████                           
> ████████████████████████████████████████████████████████████████████████████                           
> ████████████████████████████████████████████████████████████████████████████                           
> ████████████████████████████████████████████████████████████████████████████                           
> ████████████████████████████████████████████████████████████████████████████                           
> ████████████████████████████████████████████████████████████████████████████                           
> ████████████████████████████████████████████████████████████████████████████                           
> ████████████████████████████████████████████████████████████████████████████                █  █ █   ██
> ████████████████████████████████████████████████████████████████████████████       ███ █ ████ ██ ██ ███
> ████████████████████████████████████████████████████████████████████████████     █ █████ ██████████████
> ████████████████████████████████████████████████████████████████████████████  █████████████████████████
> █████████████████████████████████████████████████████████████████████████████ █████████████████████████
> ```

> 👤 **magikRUKKOLA** replied on **2025-10-07** at **05:46:47**
> 
> @Thireus made a similar quant for GLM-4.6:
> 
> ```
> Final estimate: PPL = 3.4486 +/- 0.02001
> ```
> 
> ```
> ## Summary of tensor sizes per class
> # GPU Total: 13.561 GiB (94.8%) | 14.30 GiB max, if all were q8_0 | 12.99 GiB min, if all were iq5_ks_r4
> # CPU Total: 214.783 GiB (82.0%) | 262.02 GiB max, if all were iq6_k | 168.09 GiB min, if all were iq4_ks_r4
> # GPU+CPU Total: 228.344 GiB (88.4%)
> 
> ## Summary of tensor counts and bpw per qtype
> #
> # GPU-loaded quants:
> # QTYPE         Count   BPW     Assigned GiB    % Assigned      Max GiB (all)
> # +f32          835     32.0      0.28 GiB      -               -
> # +q8_0         1       8.5       0.77 GiB      -               -
> # q8_0          19      8.5       0.20 GiB      5.9%            3.43
> # +iq6_k        373     6.625     9.82 GiB      -               -
> # iq6_k         241     6.625     2.39 GiB      89.6%           2.67
> # iq5_ks_r4     20      5.25      0.10 GiB      4.5%            2.12
> #
> # CPU-friendly quants:
> # QTYPE         Count   BPW     Assigned GiB    % Assigned      Max GiB (all)
> # iq6_k         74      6.625    71.81 GiB      27.4%           262.02
> # iq5_ks_r4     143     5.25    109.97 GiB      53.0%           207.64
> # iq4_ks_r4     53      4.25     33.00 GiB      19.6%           168.09
> #
> # -Average BPW: 5.4976
> #
> # -Notes:
> # - '+' means user-defined pre-assigned tensors, or tensor missing from csv data or f32 tensors
> # - Recipe produced on the 2025-10-06 05:17:53 UTC+0000 using Thireus' GGUF tools (https://gguf.thireus.com/)
> # - Script SHA-256: f385e17ea9998140203cc543e8e9d3635f6f1292999c58d837cc6d2d3d48b1e0
> # - Calibration dataset 'ppl_results.csv' SHA-256: 5d15ad4864a06be06db0a1e8d9fe479b475293dbf858edcf44c6e8427a7e8232
> # - tensors.bf16.map SHA-256: fa987db60ed8e9eb4348a45cb0ad630f81a97b20292ed9adfe0369ddd3ec2828
> # - tensors.bf16.map model name: GLM-4.6-THIREUS-BF16-SPECIAL_TENSOR-01760-of-01760
> # - tensors.iq6_k.map SHA-256: 0743f08065ebeeac64c52844c0a1cbde4e4e9242230d4218de7647fd46d2ba99
> # - tensors.iq6_k.map model name: GLM-4.6-THIREUS-IQ6_K-SPECIAL_TENSOR-01760-of-01760
> # - tensors.iq5_ks_r4.map SHA-256: 5c570d0b404729a5ec8d359471ac60edd86cfac80cad58999b596fcf0e61fbb0
> # - tensors.iq5_ks_r4.map model name: GLM-4.6-THIREUS-IQ5_KS_R4-SPECIAL_TENSOR-01760-of-01760
> # - tensors.iq4_ks_r4.map SHA-256: 101d062c2bbedef43ac326b7659911dfa2246fe608db2d8b86cc9d9fa1e9ed86
> # - tensors.iq4_ks_r4.map model name: GLM-4.6-THIREUS-IQ4_KS_R4-SPECIAL_TENSOR-01760-of-01760
> # - tensors.q8_0.map SHA-256: 4cdc6924a3e79e3a8df15cc40d607d01ad2d29375eb411ca423a44d437ae0c66
> # - tensors.q8_0.map model name: GLM-4.6-THIREUS-Q8_0-SPECIAL_TENSOR-01760-of-01760
> # - tensors.iq1_kt.map SHA-256: 793fce90c8b4c735e406f9ee701fc10d9e483d90fbdafdab533096ac7d9e748e
> # - tensors.iq1_kt.map model name: GLM-4.6-THIREUS-IQ1_KT-SPECIAL_TENSOR-01760-of-01760
> # - GPG signatures: DISABLED
> # - Command used:
> # ./quant_assign.py ppl_results.csv --tolerance 0.01 --cpu-irq-k 1.5 --gpu-irq-k 1.5 --gpu-assign-qtype iq6_k \
> # --cpu-tensors-max-size 215 --gpu-tensors-max-size 95% --exponential-factor 8 --cpu-tensors \
> # 'blk\.([3-9]|[1-8][0-9]|9[0-2])\.ffn_down_exps\.weight' 'blk\.([3-9]|[1-8][0-9]|9[0-2])\.ffn_up_exps\.weight' \
> # 'blk\.([3-9]|[1-8][0-9]|9[0-2])\.ffn_gate_exps\.weight' --gpu-tensors '.*' --cpu-quants iq6_k iq5_ks_r4 iq4_ks_r4 \
> # --gpu-quants iq6_k iq5_ks_r4 q8_0 --gpu-assign-tensors 'output\.weight=q8_0' --harmonize-tensors \
> # 'blk\..*\.ffn_up_exps.*,blk\..*\.ffn_gate_exps.*' --harmonization-technique 3
> ```
> 
> And again, similarly, its about +8% in prefill but (/and) better decode (+5%):
> 
> ubergarm/GLM-4.6-IQ5_K:
> ```
> main: n_kv_max = 65536, n_batch = 8192, n_ubatch = 8192, flash_attn = 1, n_gpu_layers = 99, n_threads = 12, n_threads_batch = 12
> 
> |    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
> |-------|--------|--------|----------|----------|----------|----------|
> |  8192 |   2048 |      0 |   24.715 |   331.46 |  394.268 |     5.19 |
> |  8192 |   2048 |   8192 |   28.593 |   286.51 |  420.881 |     4.87 |
> 
> 64k ctx, q8_0 KV cache with two GPUs:
>   Memory Usage:      23.5 / 24.0 GB
>   Memory Usage:      23.0 / 24.0 GB
> ```
> 
> THIREUS/GLM-4.6-5.4976bpw (R4 primarily)
> ```
> main: n_kv_max = 65536, n_batch = 8192, n_ubatch = 8192, flash_attn = 1, n_gpu_layers = 99, n_threads = 12, n_threads_batch = 12
> 
> |    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
> |-------|--------|--------|----------|----------|----------|----------|
> |  8192 |   2048 |      0 |   22.879 |   358.06 |  376.160 |     5.44 |
> |  8192 |   2048 |   8192 |   27.105 |   302.23 |  401.994 |     5.09 |
> 
> 
> 
> 64k ctx, q8_0 KV cache with two GPUs:
>   Memory Usage:      21.8 / 24.0 GB
>   Memory Usage:      21.8 / 24.0 GB
> ```

---

👤 **magikRUKKOLA** commented on **2025-10-03** at **03:24:34**

@Thireus @ubergarm 

I was thinking about an automated tool that finds the most optimal quant given the input parameters such as the RAM/VRAM limitations, the prefill/decode speed, the max context length, and the perplexity.  So that everyone would find the exact quant they want spending very little work for that.

> 👤 **magikRUKKOLA** replied on **2025-10-08** at **12:44:39**
> 
> @Thireus 
> 
> Somehow I overlooked [this] R1-0524 quant: https://raw.githubusercontent.com/Thireus/GGUF-Tool-Suite/refs/heads/main/recipe_examples/ik_harmonized_recipes/DeepSeek-R1-0528.ROOT-5.0601bpw-3.2219ppl.395GB-GGUF_13GB-GPU_382GB-CPU.90e3c2f_ea5145a.recipe
> 
> So I changed it a to use [the] R4 CPU quants:
> 
> ```
> ## Summary of tensor sizes per class
> # GPU Total: 13.733 GiB (100.0%) | 13.73 GiB max, if all were q8_0 | 13.73 GiB min, if all were q8_0
> # CPU Total: 382.156 GiB (75.8%) | 504.33 GiB max, if all were iq6_k | 323.53 GiB min, if all were iq4_ks_r4
> # GPU+CPU Total: 395.889 GiB (87.9%)
> 
> ## Summary of tensor counts and bpw per qtype
> #
> # GPU-loaded quants:
> # QTYPE         Count   BPW     Assigned GiB    % Assigned      Max GiB (all)
> # +f32          361     32.0      0.40 GiB      -               -
> # +q8_0         61      8.5       0.51 GiB      -               -
> # q8_0          185     8.5       5.54 GiB      100.0%          5.54
> # +iq5_ks_r4    366     5.25      7.29 GiB      -               -
> #
> # CPU-friendly quants:
> # QTYPE         Count   BPW     Assigned GiB    % Assigned      Max GiB (all)
> # iq6_k         24      6.625    69.56 GiB      13.8%           504.33
> # iq5_ks_r4     77      5.25    176.86 GiB      44.3%           399.66
> # iq4_ks_r4     73      4.25    135.73 GiB      42.0%           323.53
> #
> # -Average BPW: 5.0601
> #
> # -Notes:
> # - '+' means user-defined pre-assigned tensors, or tensor missing from csv data or f32 tensors
> # - Recipe produced on the 2025-10-07 22:14:34 UTC+0000 using Thireus' GGUF tools (https://gguf.thireus.com/)
> # - Script SHA-256: f385e17ea9998140203cc543e8e9d3635f6f1292999c58d837cc6d2d3d48b1e0
> # - Calibration dataset 'ppl_results.csv' SHA-256: b45b30a282e29922fc46ea4a030f1aa27df5be790c9d921cfac0fba27adc3faa
> # - tensors.bf16.map SHA-256: f264323f789d0da78bc21ccf208cbb16709c5808c9c1f939a0c78f7c03c1ece1
> # - tensors.bf16.map model name: DeepSeek-R1-0528-THIREUS-BF16-SPECIAL_TENSOR-01148-of-01148
> # - tensors.iq6_k.map SHA-256: c2b301156703fd3d360a6a9406d5079bd6625c2f7557f89c7de632df78eed822
> # - tensors.iq6_k.map model name: DeepSeek-R1-0528-THIREUS-IQ6_K-SPECIAL_TENSOR-01148-of-01148
> # - tensors.iq5_ks_r4.map SHA-256: 5916f9ee20192160667abccfb2e1f54fb110f42150eb0c9f97d19b8a52732a56
> # - tensors.iq5_ks_r4.map model name: DeepSeek-R1-0528-THIREUS-IQ5_KS_R4-SPECIAL_TENSOR-01148-of-01148
> # - tensors.iq4_ks_r4.map SHA-256: 129f784365f36bd4a0039f97674750c0128b013a5c17f7302afa29b037692ebe
> # - tensors.iq4_ks_r4.map model name: DeepSeek-R1-0528-THIREUS-IQ4_KS_R4-SPECIAL_TENSOR-01148-of-01148
> # - tensors.q8_0.map SHA-256: 8d064fb71d986348b38df6d0517ba527dd5bbd25cd1f45535971087f042f1b32
> # - tensors.q8_0.map model name: DeepSeek-R1-0528-THIREUS-Q8_0-SPECIAL_TENSOR-01148-of-01148
> # - tensors.iq1_m_r4.map SHA-256: 895e207c15f5e8a2c81f5b7061b0fe3a64a481b83c9499fb75ad7724f92d6c25
> # - tensors.iq1_m_r4.map model name: DeepSeek-R1-0528-THIREUS-IQ1_M_R4-SPECIAL_TENSOR-01148-of-01148
> # - GPG signatures: DISABLED
> # - Command used:
> # ./quant_assign.py ppl_results.csv --tolerance 0.01 --cpu-irq-k 1.5 --gpu-irq-k 1.5 --gpu-assign-qtype iq5_ks_r4 \
> # --cpu-tensors-max-size 380 --gpu-tensors-max-size 100% --exponential-factor 8 --cpu-tensors \
> # 'blk\.([3-9]|[1-5][0-9]|60)\.ffn_down_exps\.weight' 'blk\.([3-9]|[1-5][0-9]|60)\.ffn_up_exps\.weight' \
> # 'blk\.([3-9]|[1-5][0-9]|60)\.ffn_gate_exps\.weight' --gpu-tensors '.*' --cpu-quants iq6_k iq5_ks_r4 iq4_ks_r4 \
> # --gpu-quants q8_0 --gpu-assign-tensors 'blk\.([0-9]|[1-5][0-9]|60)\.attn_k_b\.weight=q8_0' --harmonize-tensors \
> # '^blk\..*\.ffn_up_exps.*,blk\..*\.ffn_gate_exps.*' --harmonization-technique 3
> ```
> 
> The PPL with 8k batches:
> ```
> Final estimate: PPL = 3.2223 +/- 0.01703
> ```
> 
> The sweep-bench for 64C 8 chaneel DDR4-3200 + 2 x RTX3090 :
> 
> ```
> ulimit -n 9999
> CUDA_VISIBLE_DEVICES="0,1" \
> /opt/ik_llama.cpp/ik_llama.cpp/build/bin/llama-sweep-bench \
>     --warmup-batch \
>     --model /opt/THIREUS/DeepSeek-R1-0528.ROOT-5.0601bpw/DeepSeek-R1-0528-THIREUS-BF16-SPECIAL_TENSOR-00001-of-01148.gguf \
>     --alias THIREUS/DeepSeek-R1-0528-5.0601bpw \
>     --ctx-size $((112 * 1024)) \
>     -b 8192 -ub 8192 \
>     --mlock \
>     --seed 3407 \
>     --temp 0.5 --top-k 0 --top-p 1.0 --min-p 0.1 --repeat-penalty 1.1 \
>     -ctk q8_0 \
>     -mla 3 -fa \
>     -amb 512 \
>     --override-tensor exps=CPU \
>     --n-gpu-layers 99 \
>     --split-mode layer \
>     --tensor-split 6,10 \
>     --threads $(grep ^cpu\\scores /proc/cpuinfo | uniq | awk '{print $4}' | xargs -I{} echo "{}-0" | bc) \
>     --host 0.0.0.0 \
>     --port 8080 \
>     --special \
>     --log-enable \
>     --logdir /var/log/ \
>     --verbosity 1 \
>     --verbose-prompt \
>     --reasoning-format none \
>     --prompt-cache "$HOME/.cache/ik_llama.cpp/prompt-cache.bin" --prompt-cache-all \
>     --slot-save-path "$HOME/.cache/ik_llama.cpp/slot.bin" \
>     --lookup-cache-dynamic "$HOME/.cache/ik_llama.cpp/slot.bin" \
>     --keep -1 \
>     --slot-prompt-similarity 0.35 \
>     --metrics
> 
> ...
> ....................................................................................................
> llama_new_context_with_model: n_ctx      = 114688
> llama_new_context_with_model: n_batch    = 8192
> llama_new_context_with_model: n_ubatch   = 8192
> llama_new_context_with_model: flash_attn = 1
> llama_new_context_with_model: mla_attn   = 3
> llama_new_context_with_model: attn_max_b = 512
> llama_new_context_with_model: fused_moe  = 0
> llama_new_context_with_model: fused_up_gate = 1
> llama_new_context_with_model: ser        = -1, 0
> llama_new_context_with_model: freq_base  = 10000.0
> llama_new_context_with_model: freq_scale = 0.025
> llama_kv_cache_init:      CUDA0 KV buffer size =  1606.51 MiB
> llama_kv_cache_init:      CUDA1 KV buffer size =  2476.71 MiB
> llama_new_context_with_model: KV self size  = 4083.19 MiB, c^KV (q8_0): 4083.19 MiB, kv^T: not used
> llama_new_context_with_model:  CUDA_Host  output buffer size =     0.49 MiB
> llama_new_context_with_model: pipeline parallelism enabled (n_copies=1)
> llama_new_context_with_model:      CUDA0 compute buffer size = 15840.03 MiB
> llama_new_context_with_model:      CUDA1 compute buffer size = 12432.03 MiB
> llama_new_context_with_model:  CUDA_Host compute buffer size =  3808.09 MiB
> llama_new_context_with_model: graph nodes  = 24343
> llama_new_context_with_model: graph splits = 214
> 
> main: n_kv_max = 114688, n_batch = 8192, n_ubatch = 8192, flash_attn = 1, n_gpu_layers = 99, n_threads = 64, n_threads_batch = 64
> 
> |    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
> |-------|--------|--------|----------|----------|----------|----------|
> |  8192 |   2048 |      0 |   35.831 |   228.63 |  248.032 |     8.26 |
> |  8192 |   2048 |   8192 |   44.152 |   185.54 |  262.106 |     7.81 |
> |  8192 |   2048 |  16384 |   52.418 |   156.28 |  275.490 |     7.43 |
> |  8192 |   2048 |  24576 |   60.804 |   134.73 |  289.005 |     7.09 |
> |  8192 |   2048 |  32768 |   69.010 |   118.71 |  300.315 |     6.82 |
> |  8192 |   2048 |  40960 |   77.502 |   105.70 |  317.015 |     6.46 |
> |  8192 |   2048 |  49152 |   86.236 |    95.00 |  328.902 |     6.23 |
> |  8192 |   2048 |  57344 |   94.761 |    86.45 |  343.068 |     5.97 |
> |  8192 |   2048 |  65536 |  104.065 |    78.72 |  354.633 |     5.77 |
> |  8192 |   2048 |  73728 |  112.820 |    72.61 |  367.935 |     5.57 |
> |  8192 |   2048 |  81920 |  121.205 |    67.59 |  381.775 |     5.36 |
> |  8192 |   2048 |  90112 |  129.894 |    63.07 |  394.891 |     5.19 |
> |  8192 |   2048 |  98304 |  138.521 |    59.14 |  407.781 |     5.02 |
> ...
> ```
> 
> system usage:
> ```
> ┌─NVIDIA GeForce RTX 3090──────────────────────────────────────────────────────────────────────────────────────────────────┐
> │ Temp: 50C  Fan: 100%  Power: 195W                                                                                        │
> │ GPU: 2055MHz (00%)  VRAM: 10251MHz (00%)  Mem: 23.7GB                                                                    │
> │ Temp [49-52]                            Fan [0-100]                             Power [142-206]                          │
> ││                     ███        ███    │██████████████████████████████████████ │                               █         │
> ││                     ███        ███    │██████████████████████████████████████ │            ██████████████████████████   │
> ││                  █ █████    ███████   │██████████████████████████████████████ │            ██████████████████████████   │
> ││                  █ █████    ███████   │██████████████████████████████████████ │█ █    █    ██████████████████████████   │
> ││  █ █ █     ██████████████████████████ │██████████████████████████████████████ │███ ██ ███ ███████████████████████████   │
> ││██████████████████████████████████████ │██████████████████████████████████████ │██████████████████████████████████████   │
> ││────────────────────────────────────── │────────────────────────────────────── │──────────────────────────────────────   │
> │ GPU Clock [1995-2055]                   VRAM Clock [10251-10251]                Util [0-85]                              │
> ││██████ ███████████████████████████████ │                                       │  █      █                               │
> ││██████ ███████████████████████████████ │                                       │█ █      █                               │
> ││██████ ███████████████████████████████ │                                       │█ █    █ █ █                             │
> ││██████ ███████████████████████████████ │                                       │█ █    █ █ █       ███       ████        │
> ││██████ ███████████████████████████████ │                                       │█ █    █ █ ███   ███████    ███████      │
> ││██████████████████████████████████████ │██████████████████████████████████████ │██████████████████████████████████████   │
> ││────────────────────────────────────── │────────────────────────────────────── │──────────────────────────────────────   │
> └──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘
> ┌─NVIDIA GeForce RTX 3090──────────────────────────────────────────────────────────────────────────────────────────────────┐
> │ Temp: 59C  Fan: 100%  Power: 243W                                                                                        │
> │ GPU: 2025MHz (27%)  VRAM: 10251MHz (18%)  Mem: 23.3GB                                                                    │
> │ Temp [59-65]                            Fan [0-100]                             Power [236-397]                          │
> ││██ █ █    █                            │██████████████████████████████████████ │ █   █                                   │
> ││██ █ █ █ ██                            │██████████████████████████████████████ │ █ ███ ██ █                              │
> ││██ █ █ █ ██                            │██████████████████████████████████████ │ █ ███ ██ █                              │
> ││██ █ █ █████ ████                      │██████████████████████████████████████ │ ██████████                              │
> ││████████████████████   ██████ ██       │██████████████████████████████████████ │████████████                             │
> ││██████████████████████████████████████ │██████████████████████████████████████ │██████████████████████████████████████   │
> ││────────────────────────────────────── │────────────────────────────────────── │──────────────────────────────────────   │
> │ GPU Clock [1710-2025]                   VRAM Clock [10251-10251]                Util [0-100]                             │
> ││    █ █ █  ███████████████████████████ │                                       │██ █ ██████                              │
> ││    █ █ █  ███████████████████████████ │                                       │██ █ ██████                              │
> ││    █ █ █  ███████████████████████████ │                                       │██ █ ██████                              │
> ││    █ █ █  ███████████████████████████ │                                       │██ █ ██████      ████       ███          │
> ││ ██████ █  ███████████████████████████ │                                       │██ █ ████████  █████████  ████████  ██   │
> ││██████████████████████████████████████ │██████████████████████████████████████ │██████████████████████████████████████   │
> ││────────────────────────────────────── │────────────────────────────────────── │──────────────────────────────────────   │
> └──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘
> RAM: 66.91 GB/s [0.58-67.11]                                                                                                
>                                                                                                                          █  
>                                                                                                  ███████████████████████████
>                                                                                                  ███████████████████████████                                                                                                 ███████████████████████████
>                                                                                                  ███████████████████████████
>                                                                                                  ███████████████████████████
>                                                                                                  ███████████████████████████
>                                                                                                  ███████████████████████████
>                                                                                                  ███████████████████████████
>                                                                                                  ███████████████████████████
>                                                                                                  ███████████████████████████
>               █                                                                                  ███████████████████████████
>               █                     █ █      █ █      █ █      █      █ █                        ███████████████████████████
>    █          █ █      █          █ █ █    █ █ █    █ █ █    █ █ █    █ █      █ █      █ █      ███████████████████████████
>    █ █ █ █    █ █      █ █ █ █    █ █ █    █ █ █    █ █ █    █ █ █    █ █      █ █      █ █      ███████████████████████████
>    █ █ █ █    █ █    █ █ █ █ █    █ █ █ █  █ █ █ █  █ █ █ █  █ █ █  █ █ █ █  █ █ █ █  █ █ █ █    ███████████████████████████
> █  █ █ █ █  █ █ █ █  █ █ █ █ █ █  █ █ █ ██ █ █ █ █  █ █ █ █  █ █ █  █ █ █ █  █ █ █ █  █ █ █ █    ███████████████████████████
> █  █ █ █ █  █ █ █ █  █ █ █ █ █ ██ █ █ █ ██ █ █ █ ██ █ █ █ ██ █ █ █ ██ █ █ ████ █ █ ████ █ █ █    ███████████████████████████
> ████████ ███████████████████████████████████████████████████████████████████████████████████████████████████████████████████
>                                                                                                                             
> Evil:Off OC:On Auto-OC:Off RAM-BW:On | Controls: [O]verclock [E]vil [R]eset [Q]uit                                          
> 
> ```

---

👤 **magikRUKKOLA** commented on **2025-10-14** at **00:40:24**

@Thireus 

Qwen3-Coder added.

```
## Summary of tensor sizes per class
# GPU Total: 17.094 GiB (100.0%) | 17.09 GiB max, if all were q8_0 | 17.09 GiB min, if all were q8_0
# CPU Total: 271.033 GiB (75.9%) | 357.13 GiB max, if all were iq6_k | 229.10 GiB min, if all were iq4_ks_r4
# GPU+CPU Total: 288.126 GiB (87.9%)

## Summary of tensor counts and bpw per qtype
#
# GPU-loaded quants:
# QTYPE         Count   BPW     Assigned GiB    % Assigned      Max GiB (all)
# +f32          311     32.0      0.23 GiB      -               -
# +q8_0         248     8.5      10.04 GiB      -               -
# q8_0          4       8.5       6.83 GiB      100.0%          6.83
#
# CPU-friendly quants:
# QTYPE         Count   BPW     Assigned GiB    % Assigned      Max GiB (all)
# iq6_k         19      6.625    36.88 GiB      10.3%           357.13
# iq5_ks_r4     98      5.25    150.73 GiB      53.3%           283.01
# iq4_ks_r4     67      4.25     83.42 GiB      36.4%           229.10
#
# -Average BPW: 5.1546
#
# -Notes:
# - '+' means user-defined pre-assigned tensors, or tensor missing from csv data or f32 tensors
# - Recipe produced on the 2025-10-13 02:12:11 UTC+0000 using Thireus' GGUF tools (https://gguf.thireus.com/)
# - Script SHA-256: f385e17ea9998140203cc543e8e9d3635f6f1292999c58d837cc6d2d3d48b1e0
# - Calibration dataset 'ppl_results.csv' SHA-256: 38d858d6842666ee42eb7450fd97c0f5bae6d79db72ade6ae5b9a5750cb7d7bb
# - tensors.bf16.map SHA-256: c95d0b5f51a24b34f981167b247dfb693649fa4ec40af103376ddb755eae8727
# - tensors.bf16.map model name: Qwen3-Coder-480B-A35B-Instruct-THIREUS-BF16-SPECIAL_TENSOR-00748-of-00748
# - tensors.iq6_k.map SHA-256: 255c8ff2a041c83c52b81615f874a728b04c0babf5135ace9218ef293cc8347a
# - tensors.iq6_k.map model name: Qwen3-Coder-480B-A35B-Instruct-THIREUS-IQ6_K-SPECIAL_TENSOR-00748-of-00748
# - tensors.iq5_ks_r4.map SHA-256: d205cc277b04c193fd5d2bf0f7962cc54a0b20607f83c532edc9f91fd6135a11
# - tensors.iq5_ks_r4.map model name: Qwen3-Coder-480B-A35B-Instruct-THIREUS-IQ5_KS_R4-SPECIAL_TENSOR-00748-of-00748
# - tensors.iq4_ks_r4.map SHA-256: 3f08223c723fed524d60108f3ee873bba6d5a1fce565611d64616651032e0a2c
# - tensors.iq4_ks_r4.map model name: Qwen3-Coder-480B-A35B-Instruct-THIREUS-IQ4_KS_R4-SPECIAL_TENSOR-00748-of-00748
# - tensors.q8_0.map SHA-256: 38b4d5922f4031af583ed27f803ee2ce98fd67468d6676b2b141bbb493be43ae
# - tensors.q8_0.map model name: Qwen3-Coder-480B-A35B-Instruct-THIREUS-Q8_0-SPECIAL_TENSOR-00748-of-00748
# - tensors.iq1_m_r4.map SHA-256: 4122072c36bdbc44872205d28d9dee2093593ec464d1d3bfb70ec40178a63e94
# - tensors.iq1_m_r4.map model name: Qwen3-Coder-480B-A35B-Instruct-THIREUS-IQ1_M_R4-SPECIAL_TENSOR-00748-of-00748
# - GPG signatures: DISABLED
# - Command used:
# ./quant_assign.py ppl_results.csv --tolerance 0.01 --cpu-irq-k 1.5 --gpu-irq-k 1.5 --gpu-assign-qtype q8_0 \
# --cpu-tensors-max-size 270 --gpu-tensors-max-size 100% --exponential-factor 8 --cpu-tensors \
# 'blk\.([0-9]|[1-5][0-9]|6[0-1]])\.ffn_down_exps\.weight' 'blk\.([0-9]|[1-5][0-9]|6[0-1])\.ffn_up_exps\.weight' \
# 'blk\.([0-9]|[1-5][0-9]|6[0-1])\.ffn_gate_exps\.weight' --gpu-tensors '.*' --cpu-quants iq6_k iq5_ks_r4 iq4_ks_r4 \
# --gpu-quants q8_0 --harmonize-tensors 'blk\..*\.ffn_up_exps.*,blk\..*\.ffn_gate_exps.*' --harmonization-technique 3
```

```
Final estimate: PPL = 5.1057 +/- 0.03269
```

---

👤 **Nexesenex** commented on **2025-10-27** at **16:19:28**

@magikRUKKOLA : Amazing work!!!

Could you please, if that's not too much of a hassle, add the sizes you collected for your graphs to your DATA SOURCES json for readability?

I invested into a new mobo/cpu set with 192GB DDR5 (+my existing 64GB of VRAM) and your data are very helpful to figure out grossly what quant I'll need to use those biggies.

> 👤 **magikRUKKOLA** replied on **2025-10-28** at **13:44:54**
> 
> @Nexesenex 
> 
> 
> Well, okay.
> You can just multiply the BPW by the number of parameters and divide it by (1024^3 * 8).
> 
> Keep in mind though that the part of the LLM in the MoE models actually ending up at GPU's VRAM, not CPU RAM.  For example:
> 
> ```json
> {"name": "THIREUS-3.1858", "bpw": 3.1858, "ppl": 3.3261, "size": 247.60, "url": "https://github.com/Thireus/GGUF-Tool-Suite/blob/main/recipe_examples/DeepSeek-R1-0528.THIREUS-3.1858bpw-3.3261ppl.249GB-GGUF_18GB-GPU_231GB-CPU.3c88ec6_027b7ff.recipe"},
> ```
> 
> You see, its about 249GB in size total but only 231GB is getting occupied in the RAM, the rest of 18GB goes into the VRAM.

> 👤 **Nexesenex** replied on **2025-10-28** at **13:54:09**
> 
> @magikRUKKOLA : Thank you for adding the data, and.. for the explanations! ^^

---

👤 **Ph0rk0z** commented on **2025-10-28** at **19:17:30**

So what should I stick to for speed? I have been going by <200GB yet that seems not to be working so well after trying a bunch of GLM REAP quants.

Despite using less memory, many reap quants were slower than the full model. Even though I have 4x3090, my PP speeds between 100-200 with smaller batch like 1024 + RTR. And yes, using high batch size can artificially pump up the number but will absolutely decimate your latency on small prompts.

When I load, I only do 32k CTX and fill the rest of the GPUs with layers or pieces of the experts. Yet IQ3_XXS pruned quant (130GB) is slower than Q4K or Q3K_XL (160-180gb). I'd try them all but I don't have the b/w to keep downloading so can't get a feel.

> 👤 **magikRUKKOLA** replied on **2025-10-28** at **21:37:28**
> 
> @Ph0rk0z 
> > So what should I stick to for speed? I have been going by <200GB yet that seems not to be working so well after trying a bunch of GLM REAP quants.
> 
> Hold on.  Hold your horses.  You were doing what?  Picking up the REAP quants?  No, it will not work this way at all.  It's been already established that one should not pick those quants because their perplexity is just not there.
> 
> https://github.com/ikawrakow/ik_llama.cpp/discussions/839#discussioncomment-14769851
> 
> Its **UNUSABLE**.  Period.
> 
> > Even though I have 4x3090, 
> 
> You're having 96GB of VRAM?  Currently I am having only three of those.  And I showed that you can run a full model of Ling-1T or Qwen3-Coder-480B with decent speed in decode.
> 
> Namely, you can run with 7 tps (**minimum**) or so with somewhat large context (20k tokens+) if you're using the speculative decoding with fp16 KV cache (**BOTH** for the target model **and** for the draft) with DDR4 system.
> 
> I mean, it's not super fast, but its quite usable.
> 
> > and fill the rest of the GPUs with layers or pieces of the experts
> 
> I see no point in this.  For the MoE models you have two options: either offload everything into VRAM (if you can, like for the case with a draft model, when you can push up to 100 tps+ in decode), or you offloading main body into RAM and the few dense layers into VRAM and having about 6-7 tps w/o spec. decoding.  You're trying to partially offload some small quant into the rest of the VRAM?
> 
> I don't think its a good idea.  The PPL of the quant you're going to use (Q3K_XL?) is not that great at all.  You'll not get the decent quality if you're using it as a target model.  And because you're trying to speed it up by offloading some of the layers into the free GPU VRAM space you'll have no space for the draft model at all.
> 
> The question is -- why you're using it this way?  Isn't it better to set up the inference with spec. decoding?
> 
> What are the decode speed you're getting while offloading some layers into the GPU?  I am getting from 10% to 100% boost in decode with spec. decoding.  So its from about 6 tps up to 12 tps+ (with 20 ctx context).

> 👤 **magikRUKKOLA** replied on **2025-10-28** at **21:45:50**
> 
> @Ph0rk0z 
> > I'd try them all but I don't have the b/w to keep downloading so can't get a feel.
> 
> It depends what you're trying to run.
> 
> For example I really like DeepSeek-R1 model.  As far as I can see, since May of 2025, there were no model which would compete with R1 in reasoning.  And there is no way to run R1 with spec. decoding.  Any of LLAMA or Qwen distill stuff is just not there.  So you have two options.
> 
> 1.)
> 
> ```
> {"name": "THIREUS-3.5652", "bpw": 3.5652, "ppl": 3.2734, "size": 284.90, "url": "https://github.com/Thireus/GGUF-Tool-Suite/blob/main/recipe_examples/DeepSeek-R1-0528.THIREUS-3.5652bpw-3.2734ppl.278GB-GGUF_14GB-GPU_264GB-CPU.3c88ec6_9b5660b.recipe"},
> ```
> 
> 3.5652 bpw.  Its decent and fast if you prefer speed.
> 
> 2.) 
> 
> ```
>  {"name":"THIREUS-5.0601","bpw":5.0601,"ppl":3.2223,"size": 397.12, "url":"https://github.com/ikawrakow/ik_llama.cpp/discussions/715#discussioncomment-14625973"},
> ```
> 
> It's has 3.2223 PPL while the Q8_0 baseline has 3.2130.  So it's almost there while being much smaller than Q8_0.
> 
> That's the quant you'll use for the best possible quality while keeping the whole setup under 512GB of RAM.
> 
> 
> etc. etc.

> 👤 **magikRUKKOLA** replied on **2025-10-28** at **22:08:28**
> 
> @Ph0rk0z 
> 
> > Even though I have 4x3090, my PP speeds between 100-200 with smaller batch like 1024 + RTR.
> 
> Eh.  Meh.  You do understand that PP speed it almost directly proportional to the batch size, right?  I found that for the best PP speed you'd be better off using 8k batches ( -b and -ub ).  I am getting from 200 tps up to 300 tps with such setup.  I can't complain -- the prefill speed is marvelous!  Especially when the prefix caching works great (you can even dump and restore it into/from the SSD).  The bottleneck is the decode speed.  And we have only two options -- either upgrading the RAM (or offloading everything into VRAM), or using the spec. decoding.
> 
> For example I am building the DDR5 system right now with Xeon QYFS 56C CPU and 4800 MT/s DDR5.  I know for sure it will be about 50% better than DDR4 3200 MT/s.  The main idea from here is to find the right small quant for the spec. decoding to boost up the decode speed even further (I am sure its possible to reach 20 tps+).  There is simply no other way to speed it up really. :)

> 👤 **Ph0rk0z** replied on **2025-10-28** at **22:19:46**
> 
> If I don't offload to GPU, I get more like 3-4t/s because of numa and using xeon scalable without AMX or full-bore AVX-512 ML extensions. My memory is only 2666 and while I see a bench of 230GB/s, in PCM-MEMORY speeds don't really use it.
> 
> Here is some more stuff I ran today:
> 
> DeepSeek-R1-0528-UD-IQ1_S
> 
> |    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
> |-------|--------|--------|----------|----------|----------|----------|
> |  4096 |   1024 |      0 |   26.285 |   155.83 |  102.492 |     9.99 |
> |  4096 |   1024 |   4096 |   28.842 |   142.01 |  105.654 |     9.69 |
> 
> but in chat becomes:
> ```
> 
> prompt eval time     =   19857.52 ms /  1286 tokens (   15.44 ms per token,    64.76 tokens per second)
> print_timings] generation eval time =    5863.67 ms /    59 runs   (   99.38 ms per token,    10.06 tokens per second)
> ```
> 
> 
> 
> DeepSeek-V3-0324-UD-IQ2_XXS
> 
> |    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
> |-------|--------|--------|----------|----------|----------|----------|
> |  1024 |    256 |      0 |   21.635 |    47.33 |   23.924 |    10.70 |
> |  1024 |    256 |   1024 |   22.150 |    46.23 |   24.228 |    10.57 |
> 
> 
> GLM
> |    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
> |-------|--------|--------|----------|----------|----------|----------|
> |  1024 |    256 |      0 |    8.547 |   119.81 |   17.082 |    14.99 |
> |  1024 |    256 |   1024 |    8.513 |   120.29 |   17.350 |    14.75 |
> |  1024 |    256 |   2048 |    8.815 |   116.17 |   17.926 |    14.28 |
> |  1024 |    256 |   3072 |    8.634 |   118.59 |   18.571 |    13.79 |
> 
> 
> Haven't tried speculative decoding yet because it wasn't working until recently. My use case is chat so lots of shorter messages back and forth. Don't need Q8_0 quality and I've only got 384GB of ram.
> 
> What I stick in vram is ffn* layers, the up/down/gate, same as you send to sysram.
> 
> for example:
> ```
> 
>     -ot "blk\.(8|9|10|11|12)\.ffn_.*(exps).=CUDA0" \
>     -ot "blk\.(13|14|15|16|17|18|19)\.ffn_.*(exps).=CUDA1" \
>     -ot "blk\.(20|21|22|23|24|25|)\.ffn_.*(exps).=CUDA2" \
>     -ot "blk\.(26|27|28|29|30|31|)\.ffn_.*(exps).=CUDA3" \
>     -ot "blk\.(33)\.ffn_(up)_exps.=CUDA2" \
>     -ot "blk\.(33)\.ffn_(gate)_exps.=CUDA2" \
>     -ot "blk\.(34)\.ffn_(down)_exps.=CUDA3" \
>     -ot "ffn_.*_exps.=CPU"
> ```

> 👤 **magikRUKKOLA** replied on **2025-10-28** at **22:34:28**
> 
> @Ph0rk0z 
> > I have been going by <200GB yet that seems not to be working so well after trying a bunch of GLM REAP quants.
> 
> Oh you're having 192GB of RAM or so?  That's the difficult situation to be in.
> I chose older AMD Threadripper PRO CPUs to do the job because they can reach around from 71 GB/s up to 115 GB/s depending on the number of CCDs on chip.  That is, cheap CPUs like 3945wx have only 2 CCDs (71 GB/s).  The full CPUs like 3995wx have 8 CCDs (115 GB/s).  For the curiosity, I've ordered the 39[7]5wx which has 4 CCD.  I am sure the real performance is going to be somewhere in the middle.
> So the CPU itself is a limiting factor.  For 150 EUR you'll have 71 GB/s.  For 600 EUR, somewhere between 71 GB/s and 115 GB/s and for the full 3995wx (1600 EUR?) -- the 115 GB/s.
> 
> The DDR4 ECC RAM is really cheap nowadays (700 EUR per 8 sticks of 2666 MT/s easily overclockable to the full 3200 MT/s).
> 
> I can't fathom how can you get more.  Suppose you're having a regular server motherboard like MC62-G40 or, say, ASROCK WRX80.  The motherboard such as that have 7 PCIe.  Only four of them are full 16x.  How many GPUs can you connect to it?  Actually, four (plus two NVLINKs if you're using RTX 3090).  Then, to connect more, you'll have to use risers and bifurcators.  How many would you able to connect?  Suppose, with PCIe gen. 4 with 4x speed ... 
> 
> ```
> echo "4*4+3*2"|bc
> 22
> ```
> 
> 22 GPUs (24 GB VRAM each) -> 528 GB of VRAM.
> 
> Well ... its threoretically feasible but ... its 22 GPUs !!!  imagine the maintenance hell and the number of radiators for such setup.  Its 8.8 kw in peak.  So it's unfortunately seems like madness.  That's why I am saying there simply no other way but to use spec. decoding and up to 4 GPU per workstation.

> 👤 **magikRUKKOLA** replied on **2025-10-28** at **22:41:26**
> 
> @Ph0rk0z 
> > My memory is only 2666 and while I see a bench of 230GB/s, in PCM-MEMORY speeds don't really use it.
> 
> Wait.  What?
> 
> 230 GB/s?!  If not too much of a hassle, can you please run this:  https://github.com/ikawrakow/ik_llama.cpp/discussions/818#discussion-8986330  to actually benchmark the real BW?

> 👤 **magikRUKKOLA** replied on **2025-10-28** at **22:54:25**
> 
> @Ph0rk0z 
> > DeepSeek-V3-0324-UD-IQ2_XXS
> 
> Eh.  Meh.  ... These quants are not that great.  They are making some stupid mistakes once in a while -- like syntax errors while they are producing the code, for example.  So you're not going to be able to just pick the code they are producing and run it because of that.  You'd rather would want to go for something like 4 bpw quant to have a decent quality.  But you're saying you got 384GB of RAM?  You should be able to get it.  Just pick up the heaviest quant possible which would fit into the RAM and overclock your GPUs.  If you can, use spec. decoding (Qwen3-Coder and Ling-1T).  That's about it.

> 👤 **magikRUKKOLA** replied on **2025-10-28** at **23:05:47**
> 
> @Ph0rk0z 
> 
> > DeepSeek-V3-0324-UD-IQ2_XXS
> > S_PP t/s: 47.33
> 
> 47.33 tps in prefill???
> 
> Something is really wrong here!  I am getting ... let me check real quick with my Lenovo Thinkstation P620 setup. :)

> 👤 **Ph0rk0z** replied on **2025-10-28** at **23:17:20**
> 
> Well it was a good call on speculative decoding. I used jukeofyork model with R1 and at least the sweep bench go up. I think +1.5t/s and a little bit of prompt processing. 
> 
> |    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
> |-------|--------|--------|----------|----------|----------|----------|
> |  4096 |   1024 |      0 |   26.300 |   155.74 |   88.997 |    11.51 |
> 
> You have to take a few layers off GPU but I think it's not bad. In benchmarks it makes up for it.
> 
> Since the model is cached, I tried no layers on GPU:
> 
> |    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
> |-------|--------|--------|----------|----------|----------|----------|
> |  4096 |   1024 |      0 |   38.169 |   107.31 |  154.445 |     6.63 |
> 
> 
> And with spec decoding:
> 
> |    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
> |-------|--------|--------|----------|----------|----------|----------|
> |  4096 |   1024 |      0 |   38.083 |   107.55 |  134.463 |     7.62 |
> 
> 
> Instead of that benchmark, I use MLC from intel. 
> 
> ```
> ALL Reads        :	236986.0	
> 3:1 Reads-Writes :	212831.2	
> 2:1 Reads-Writes :	210467.8	
> 1:1 Reads-Writes :	200761.0	
> Stream-triad like:	188303.0
> ```
> 
> During inference on this model I see 32gb/s only. Maybe slightly more when it was all on ram. Meanwhile if I load something to fastllm and let it use numa, I see all b/w. Maybe pcm-memory isn't super accurate?
> 
> But the issue still remains, regardless of quant quality. Some just work much faster. Maybe I could use some Q4 like this and maintain the speed
> GLM:
> IQ3_XXS = 90t/s PP, 11t/s TG (113GB)
> Q4_K = 130t/s PP, 13t/s TG (162GB)
> Q3K_XL 120t/s PP ~15t/s TG (158GB)
> 
> 113GB quant has very little offloaded and yet is super slow.

> 👤 **Ph0rk0z** replied on **2025-10-28** at **23:19:32**
> 
> > I can't fathom how can you get more. Suppose you're having a regular server motherboard like MC62-G40 or, say, ASROCK WRX80.
> 
> I have a supermicro and PLX. Technically it should fit 8 GPU at x16 but 2 of my slots are busted. With P2P drivers, interconnect speed isn't too bad.

> 👤 **Ph0rk0z** replied on **2025-10-28** at **23:46:48**
> 
> Whelp.. in sweep bench I saw a gain so either it never used it? or 100% acceptance rate. In server with actual messages it cuts t/s in half.

> 👤 **magikRUKKOLA** replied on **2025-10-29** at **00:27:14**
> 
> @Ph0rk0z
> > I used jukeofyork model with R1 and at least the sweep bench go up. I think +1.5t/s and a little bit of prompt processing.
> 
> Wait a sec.  The speed-up in decode with spec. decoding of a draft model under 1B parameters?  llama-sweep-bench uses the synthetic data in the test.  Let me quote:
> 
> https://github.com/ikawrakow/ik_llama.cpp/discussions/839#discussioncomment-14717849
> ```
> There is no support for a draft model in llama-bench or llama-sweep-bench. But these tools use random data, so even if one would make the effort to support a draft model, results would not be representative for real world usage (where we know that the success rate of the draft model and corresponding speedup (or slowdown) is highly context dependent). So, then, to have a proper benchmark one would need to start adding the ability to load and use specific contexts, which would make things way more complicated.
> ```
> 
> So I am not sure what you were measuring with such spec. decoding.
> 
> What I was doing to measure the real spec. decoding speed is prefilling of https://github.com/Thireus/GGUF-Tool-Suite/blob/main/quant_assign.py and asking the question "explain the code". :)  The response is usually about 2k tokens.  Pretty good test in my opinion.

> 👤 **magikRUKKOLA** replied on **2025-10-29** at **00:28:35**
> 
> @Ph0rk0z 
> > During inference on this model I see 32gb/s only.
> 
> Yeah, something along the lines I can see for the prefill phase only.  For the decode its more like 110 GB/s.   (that is, the max of what the real BW test shows me).

> 👤 **magikRUKKOLA** replied on **2025-10-29** at **00:34:16**
> 
> > in sweep bench I saw a gain so either it never used it? or 100% acceptance rate.
> 
> Honestly, I don't know how you got these results and what they were measuring.
> 
> >  In server with actual messages it cuts t/s in half.
> 
> Yes, this sounds more like it.  In case you're trying to use ~1B model as a draft model for spec decoding for such behemoth as DeepSeek-R1 you'll get about that -- that is, the acceptance rate less than 50% and decrease in speed in half or so.
> 
> That's exactly what I was saying -- you'd have to use the model for spec. decoding which shows **AT LEAST** 60% of the acceptance rate.  Otherwise you'll get your speed cut down in half of so.  Honestly, I don't know why such models exists at all ( that is, https://huggingface.co/jukofyork/DeepSeek-R1-DRAFT-0.6B-v3.0-GGUF ) -- they simply do not work.

> 👤 **Ph0rk0z** replied on **2025-10-29** at **01:02:26**
> 
> I got the results by running the same speculative parameters in sweep bench as server. Is there any way to see acceptance rate in server? Do I have to put on verbose?
> 
> Highest pcm-memory I see steadily is maybe 62GB on GLM. It's during token gen afaik, the cards can communicate P2P during prompt processing. I see large PCIE traffic then.
> 
> There is one of those models for mistral-large so I'll see if I also get a speed cut in exllama. What else would work for deepsek? I could try using lite, its 8GB. Maybe there is some calculus where spec decoding > layers on GPU.

> 👤 **magikRUKKOLA** replied on **2025-10-29** at **02:27:19**
> 
> @Ph0rk0z 
> > I got the results by running the same speculative parameters in sweep bench as server. Is there any way to see acceptance rate in server? Do I have to put on verbose?
> 
> Of course there is a way.  Yeah, you have to put the verbose-something.  I believe these:
> ```
>  --verbosity 2 --verbose-prompt
> ```
> 
> Take a look at the bash scripts here:  https://github.com/ikawrakow/ik_llama.cpp/discussions/839#discussioncomment-14763959
> 
> > What else would work for deepsek?
> 
> Possibly the distill deepseek like ~~Qwen-8B~~ Qwen-14B ?  But I haven't tried it yet.

> 👤 **Ph0rk0z** replied on **2025-10-29** at **12:24:34**
> 
> Sadly the distils don't match special tokens. I tried a larger qwen and it didn't work.

> 👤 **magikRUKKOLA** replied on **2025-10-29** at **15:31:11**
> 
> @Ph0rk0z 
> > Possibly the distill deepseek like ~Qwen-8B~ Qwen-14B ? But I haven't tried it yet.
> 
> No luck.

---

👤 **magikRUKKOLA** commented on **2025-10-29** at **02:31:51**

@Ph0rk0z 

related to: https://github.com/ikawrakow/ik_llama.cpp/discussions/715#discussioncomment-14808884

sweep-bench 4k batches with THIREUS-R1-3.5652bpw, 124k ctx, two rtx 3090:

```
 /opt/ik_llama.cpp/ik_llama.cpp/build/bin/llama-sweep-bench --warmup-batch --model /mnt/data/opt/THIREUS-R1-3.5652bpw/DeepSeek-R1-0528-THIREUS-BF16-SPECIAL_TENSOR-00001-of-01148.gguf --alias THIREUS/DeepSeek-R1-0528-3.5652bpw --ctx-size 126976 --temp 0.5 --top-k 0 --top-p 1.0 --min-p 0.1 --repeat-penalty 1.0 -ctk q8_0 -mla 3 -amb 512 -b 4096 -ub 4096 --split-mode layer --tensor-split 7,8 --main-gpu 1 --override-tensor exps=CPU --n-gpu-layers 99 --threads 64 --host 0.0.0.0 --port 8080 --lookup-cache-dynamic /mnt/data/ik_llama.kv.dump --log-enable --logdir /var/log/ --jinja --special --verbose-prompt --verbosity 2
```

```
main: n_kv_max = 126976, n_batch = 4096, n_ubatch = 4096, flash_attn = 1, n_gpu_layers = 99, n_threads = 64, n_threads_batch = 64

|    PP |     TG |   N_KV |   T_PP s | S_PP t/s |   T_TG s | S_TG t/s |
|-------|--------|--------|----------|----------|----------|----------|
|  4096 |   1024 |      0 |   22.752 |   180.03 |   97.566 |    10.50 |
|  4096 |   1024 |   4096 |   24.661 |   166.09 |  100.495 |    10.19 |
|  4096 |   1024 |   8192 |   26.795 |   152.86 |  104.519 |     9.80 |
|  4096 |   1024 |  12288 |   29.025 |   141.12 |  107.518 |     9.52 |
|  4096 |   1024 |  16384 |   31.115 |   131.64 |  110.798 |     9.24 |
|  4096 |   1024 |  20480 |   33.330 |   122.89 |  114.090 |     8.98 |
|  4096 |   1024 |  24576 |   35.468 |   115.48 |  116.913 |     8.76 |
|  4096 |   1024 |  28672 |   37.656 |   108.77 |  120.709 |     8.48 |
|  4096 |   1024 |  32768 |   40.403 |   101.38 |  123.557 |     8.29 |
|  4096 |   1024 |  36864 |   42.638 |    96.07 |  127.204 |     8.05 |
...
```

So the prefill is about 150 tps.  I have no idea why you're getting only 40-something.

![lenovo-3995wx-2x3090rtx](https://github.com/user-attachments/assets/0381824f-61e4-4799-ac9b-ef161187a8d6)

```
nvoc -R
Detected 2 NVIDIA GPU(s)

NVOC System Report
==================
System: Linux lenovo 6.16.12+deb14+1-amd64 x86_64
Driver Version: 580.95.05

GPU 0: NVIDIA GeForce RTX 3090
------------------------------------------------
  PCI Bus ID:        00000000:41:00.0
  VBIOS Version:     94.02.4B.00.0B
  Persistence Mode:  Enabled
  Core Temperature:  57°C
  Power Usage:       214W
  Current Power Limit: 400W
  Power Limits:      Default: 350W, Min: 100W, Max: 400W
  GPU Clock:         2055 MHz
  VRAM Clock:        10251 MHz
  GPU Utilization:   18%
  VRAM Utilization:  12%
  Memory Usage:      23.6 / 24.0 GB
  Applied Offsets:   GPU: 100 MHz, VRAM: 1500 MHz

GPU 1: NVIDIA GeForce RTX 3090
------------------------------------------------
  PCI Bus ID:        00000000:61:00.0
  VBIOS Version:     94.02.4B.00.0B
  Persistence Mode:  Enabled
  Core Temperature:  43°C
  Power Usage:       192W
  Current Power Limit: 400W
  Power Limits:      Default: 350W, Min: 100W, Max: 400W
  GPU Clock:         2025 MHz
  VRAM Clock:        10251 MHz
  GPU Utilization:   20%
  VRAM Utilization:  14%
  Memory Usage:      23.5 / 24.0 GB
  Applied Offsets:   GPU: 100 MHz, VRAM: 1500 MHz


Peer-to-Peer (P2P) Support Matrix:
=================================
GPU 0 -> GPU 1: Supported
GPU 1 -> GPU 0: Supported

Report generated at: Wed Oct 29 02:34:52 2025
==================
```

> 👤 **Ph0rk0z** replied on **2025-10-29** at **12:28:29**
> 
> You have better CPU and no numa? I got close to that on the IQ1 quant. If I had the b/w I'd try the script to see what kind of custom quant I could make.

> 👤 **magikRUKKOLA** replied on **2025-10-29** at **12:47:33**
> 
> > You have better CPU and no numa?
> 
> Well, I've got only one node:
> 
> ```bash
> numactl --hardware
> available: 1 nodes (0)
> node 0 cpus: 0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 50 51 52 53 54 55 56 57 58 59 60 61 62 63 64 65 66 67 68 69 70 71 72 73 74 75 76 77 78 79 80 81 82 83 84 85 86 87 88 89 90 91 92 93 94 95 96 97 98 99 100 101 102 103 104 105 106 107 108 109 110 111 112 113 114 115 116 117 118 119 120 121 122 123 124 125 126 127
> node 0 size: 515597 MB
> node 0 free: 204789 MB
> node distances:
> node     0
>    0:   10
> ```
> 
> And, importantly as well, I found that this speed the things up a little bit.
> 
> ```bash
> cat /proc/cmdline | grep -Eo 'mitigations=[^ ]+'
> mitigations=off
> ```
> 
> I also got a better RAM.  That seems to be a limiting factor.
> 
> ```
> Handle 0x0026, DMI type 17, 92 bytes
> Memory Device
>         Array Handle: 0x0022
>         Error Information Handle: 0x0025
>         Total Width: 72 bits
>         Data Width: 64 bits
>         Size: 64 GB
>         Form Factor: DIMM
>         Set: None
>         Locator: DIMM5
>         Bank Locator: BANK4
>         Type: DDR4
>         Type Detail: Synchronous Registered (Buffered)
>         Speed: 3200 MT/s
>         Manufacturer: Samsung
>         Serial Number: 24BC5FD1
>         Asset Tag: Not Specified
>         Part Number: M393A8G40BB4-CWE
>         Rank: 2
>         Configured Memory Speed: 3200 MT/s
>         Minimum Voltage: 1.2 V
>         Maximum Voltage: 1.2 V
>         Configured Voltage: 1.2 V
>         Memory Technology: DRAM
>         Memory Operating Mode Capability: Volatile memory
>         Firmware Version: Unknown
>         Module Manufacturer ID: Bank 1, Hex 0xCE
>         Module Product ID: Unknown
>         Memory Subsystem Controller Manufacturer ID: Unknown
>         Memory Subsystem Controller Product ID: Unknown
>         Non-Volatile Size: None
>         Volatile Size: 64 GB
>         Cache Size: None
>         Logical Size: None
> 
> ...
> ```

> 👤 **Ph0rk0z** replied on **2025-10-29** at **13:50:00**
> 
> Its funny because putting everything on one node, where I should get a good 100gb/s, doesn't help either. Something in llama.cpp causes bottleneck. I tried putting everything on CPU, one CPU, eliminating the QPI and PCIE but no go.
> 
> I can also turn off numa and then I get slightly better speeds until memory gets fragmented. Mitigations I already disabled.
> 
> It could also be reporting but fastllm shows full b/w there when inferenceing. OTOH doing the math, I should not be seeing speeds this high for output if it matched pcm-memory.
> 
> I did MLC between 2400/2666/2933 since I am able to overclock a bit due to older microcode. The difference isn't really that much between them.

> 👤 **magikRUKKOLA** replied on **2025-10-29** at **14:02:26**
> 
> @Ph0rk0z 
> 
> > I should not be seeing speeds this high for output if it matched pcm-memory.
> 
> Okay cool.  But what about:
> 
> ```
> sudo apt install libopenmpi-dev
> wget https://www.cs.virginia.edu/stream/FTP/Code/stream.c
> gcc -O3 -fopenmp -DSTREAM_ARRAY_SIZE=100000000 -DNTIMES=100 stream.c -o stream
> ./stream
> ```
> 
> ??
> 
> 
> These are my results for Lenovo P620:
> ```
> Function    Best Rate MB/s  Avg time     Min time     Max time
> Copy:          108629.1     0.015497     0.014729     0.021233
> Scale:         102973.5     0.016299     0.015538     0.023051
> Add:           109444.0     0.022571     0.021929     0.028145
> Triad:         111049.8     0.022543     0.021612     0.030841
> ```
> 
> And this one is for the same CPU but with overclocked to 3200 MT/s RAM (from 2666 MT/s):
> 
> ```
> Function    Best Rate MB/s  Avg time     Min time     Max time
> Copy:          112820.2     0.016658     0.014182     0.022213
> Scale:         106723.6     0.017186     0.014992     0.025245
> Add:           113862.2     0.023645     0.021078     0.029408
> Triad:         115763.5     0.023327     0.020732     0.029096
> ```
> 
> The results are better because of better timings.
> 
> You're saying you got more than 200 GB/s?  This is very strange.

> 👤 **Ph0rk0z** replied on **2025-10-29** at **14:11:33**
> 
> On stream I get this
> 
> ```
> 
> Function    Best Rate MB/s  Avg time     Min time     Max time
> Copy:          172765.1     0.010152     0.009261     0.017204
> Scale:         137172.4     0.012634     0.011664     0.027495
> Add:           148974.1     0.016898     0.016110     0.029151
> Triad:         150104.8     0.017074     0.015989     0.024037
> ```
> 
> Imo, its a weaker benchmark than MLC.

> 👤 **magikRUKKOLA** replied on **2025-10-29** at **15:19:28**
> 
> @Ph0rk0z 
> 
> > Imo, its a weaker benchmark than MLC.
> 
> Thanks for the info!
> 
> The reason I am  using this benchmark is that the steady RAM BW during inference is exactly the same as I am seeing in this benchmark.  That is, in my case it would not go above 115 GB/s anyways during decode.
> 
> But that did not translates to your case.  Which is sad.

> 👤 **Ph0rk0z** replied on **2025-10-29** at **16:37:47**
> 
> It is not very numa aware so it probably behaves kind of like llama.cpp. I had also assumed to see higher bandwidth used when I put more on CPU but it turned out to not work that way. Maybe I can find another tool besides pcm-memory to do the measurements.

> 👤 **magikRUKKOLA** replied on **2025-10-29** at **23:58:42**
> 
> @Ph0rk0z 
> 
> Hey bro!  The way you thinking is beautifully strict and nice!  Please keep me subscribed!
> 
> You see, the maintainer of ik_llama.cpp and all of the other followers are basically saying that decode is limited by the RAM BW. :)  So you're showing that your RAM BW is far much superior to what my average single CPU workstation shows.  I mean ... I am having 115 GB/sec or so.  Your system, alternatively, shows 150 GB/sec.  And at the same time you're not getting the speed you have to.  This is absolute bananas.  You do understand it, right?
> 
> First off, can you show your **EXACT** hardware specs. etc. from where you're getting 150 GB/sec etc.?
> 
> If what you're saying is indeed true, I think everyone have to pursue the goal of even faster inference via the usage of the 2+ CPUs etc.  I think we should figure out where exactly is the bottleneck.  ~~Once its done the fix should be very easy.~~  Ah fuck.  Nothing is easy in this World. ...  Please make sure to provide more debug info.  Where did you get your motherboard from?  What about the CPUs and the RAM sticks etc.?  What is your BIOS version?  What are the RAM timings? etc.  Please provide more info.

> 👤 **Ph0rk0z** replied on **2025-10-30** at **10:49:03**
> 
> Oh, I get the full 200+, it shows inferencing qwen 235b with fastLLM. This benchmark isn't numa aware or optimized. 30-62gb/s doesn't match PCIE, QPI or any interface for a bottleneck.
> 
> The board is: https://www.supermicro.com/en/products/motherboard/X11DPG-OT-CPU
> CPU is: Intel Xeon QQ89 x2
> Ram sticks are: Samsung 32gb M393A4K40CB1-CRC4Q
> 
> Timings I don't know but the sticks are overclocked from 2400 -> 2666. 2933 was not stable either because of the CPU or the memory itself. Likely it's whatever JDEC is for 2666.
> 
> The fix is super not easy. Basically needs tensor parallel between CPU so they only calculate and access local memory. I really do need to check with other tools though in case the access patterns of pcm-memory cause it to under-report speeds.

> 👤 **ikawrakow** replied on **2025-10-30** at **10:54:23**
> 
> > Oh, I get the full 200+, it shows inferencing qwen 235b with fastLLM
> 
> Based on your other comments, that would mean that you are getting 3-4X better TG with fastLLM than with `ik_llama.cpp`.  Why are you even bothering to use `ik_llama.cpp`?

> 👤 **magikRUKKOLA** replied on **2025-10-30** at **13:09:57**
> 
> @Ph0rk0z 
> > Oh, I get the full 200+, it shows inferencing qwen 235b with fastLLM.
> 
> With INT4 quants?

> 👤 **Ph0rk0z** replied on **2025-10-30** at **13:24:00**
> 
> AWQ quants. That's what it originally used. Situation is much more complicated though. fastLLM can't do numa and put layers on the GPU at the same time. You can only use GPU for attention/kv and then numa for T/G. The reported b/w in pcm-memory is higher, which is more of a "yes this b/w is available and would speed things up" kind of thing. Rather then _OMG it's soo much better._
> 
> fastllm also lags on sampler/model support and overall t/s is higher in IK using hybrid loading. plus the whole thing is mostly chinese and it's error messages are "segmentation fault" if you mess something up. 
> 
> When mainline was faster on offloaded gpt-oss you weren't all like "just use mainline".

> 👤 **magikRUKKOLA** replied on **2025-10-30** at **16:32:08**
> 
> @Ph0rk0z 
> > CPU is: Intel Xeon QQ89 x2
> 
> Your CPU only support PCIe 3.0?
> 
> ```
> PCI Express Revision 
> 3.0
> Max # of PCI Express Lanes 
> 48
> ```
> 
> Its from Scalable family, right?
> 
> If so, this is likely the cause of your slower prefill.

> 👤 **Ph0rk0z** replied on **2025-10-30** at **17:44:59**
> 
> Yep, its scalable without ML extensions. But the CPU has PLX for the PCIE slots (X9DRG-O-PCIE) which supports 4 x16 per side.
> 
> In theory a bottleneck would be solved there via using a single GPU or only 2 GPU. Might be the next thing to try. Problem is I don't ever see full speed, even if I simply put it on all CPU.
> 
> <img width="962" height="520" alt="ik_cpu" src="https://github.com/user-attachments/assets/8b905d4d-4634-433c-8976-4f56afb396dc" />
> <img width="977" height="411" alt="ik_cpu2" src="https://github.com/user-attachments/assets/c2e580de-9482-40d2-b3ef-6bb1defbd58e" />
> 
> There's 2 full x16 to each CPU which then are subdivided. None of the speeds I get match up with either QPI or PCIE limit. Neither during prefill or during generation. When I run NCCL wan distributed I see much more PCIE traffic than during ik_llama inference or really any tensor parallel backend. Hence I've been tearing my hair out. Fully offloaded models like mistral large get normal prefill speeds like yours.
> 
> Maybe there are some better profiler tools to see everything at once.
> 
> edit: I tested 1x, 2x GPU vs 4x. Speed goes up to 44GB/s from 37GB/s with 2x GPU but prompt processing goes down. From 158 to 118t/s. 1x GPU also drops.

> 👤 **magikRUKKOLA** replied on **2025-10-30** at **20:49:54**
> 
> > When I run NCCL wan distributed I see much more PCIE traffic than during ik_llama inference or really any tensor parallel backend.
> 
> Can you please be a little bit more specific?  You're running what?
> 
> What does your p2pBandwidthLatencyTest says BTW?
> 
> > There's 2 full x16 to each CPU which then are subdivided.
> 
> Full x16 of PCIe 3.0?  Well, this is very sad.  Any of the motherboards for AMD Threadripper (even Lenovo) have 4 x16 PCIe 4.0 slots.  So it looks like even you can get superior RAM BW with two CPUs, your PCIe lanes are struggling.
> 
> You basically need a code with NUMA optimizations etc to use what you got.  It doesn't look like you're going to get any help here.  The ik_llama.cpp maintainter, Mr. Ivan, many times explicitly states that there is just not enough hours in a day to accommodate all the various hardware configurations.
> 
> Your situation is sad.  I don't think I can help either because I chose simply to upgrade to DDR5 with Xeon QYFS.
> 
> P.S.: lol last time I checked the Xeon QYFS was 120 USD at ebay.  So one can ship it directly from China without taxes lol.  The motherboards from these CPU are not that expensive.  Its about 1k EUR.  And, like DDR5 4800 MT/s 8 sticks will go for about 1.6k EUR or so.  So its 3k EUR basically for a base, without the GPUs.  Its very cheap and this system will beat your setup at any time.  You gonna have to upgrade as well, bro.

> 👤 **magikRUKKOLA** replied on **2025-10-30** at **21:28:05**
> 
> > edit: I tested 1x, 2x GPU vs 4x. Speed goes up to 44GB/s from 37GB/s with 2x GPU but prompt processing goes down. From 158 to 118t/s. 1x GPU also drops.
> 
> The PP drops?  For what?  For 1-bit quant?  The DeepSeek-R1-0528-UD-IQ1_S?  Bro, this is simply sad.
> 
> I am sure it should be theoretically possible to code your way out of it.  I would use Ling-1T and Deepseek-R1 and Deepseek-V3.1-Terminus for such a job, but you see, I am too lazy for that, especially given that I don't have any two-socket motherboards etc.

> 👤 **Ph0rk0z** replied on **2025-10-31** at **11:15:31**
> 
> I can see PCIE bandwidth inside nvtop and it's not maxed out. In WAN (the video model) it uses more. I know the upgrade path is DDR5 but $3K is not cheap, plus in the US there is tariff. A lot of people have dual socket epyc and other such boards which will have similar issues. Don't know either how much have_fancy_SIMD is worth on processors. 
> 
> I got my system in early 2023 and mainly expanded. It was before MoE times. IK simply doesn't have a dual CPU or multi-gpu setup and that's why we don't have tensor parallel for CPU or GPU. Only hope is mainline or porting the code from fastllm, which is partly why I keep bringing it up. They already took code from here to support GGUF and have numa code.

> 👤 **magikRUKKOLA** replied on **2025-10-31** at **12:22:15**
> 
> @Ph0rk0z 
> > I can see PCIE bandwidth inside nvtop and it's not maxed out. 
> 
> What about the latency?  You see, the tool from nvidia-toolkit I was mentioning, it provides such info.  nvtop doesn't.
> 
> > IK simply doesn't have a dual CPU or multi-gpu setup and that's why we don't have tensor parallel for CPU or GPU.
> 
> I don't think the absence or the presence of a certain thing can do a dent in this regard.  For example I do have a hardware to build the DDR5 system but I simply do not have a time to build it etc..  Also, anyone with some spare machines (I only have multi-GPU setups) can provide an SSH access etc.  but I never seen any such requests from the IK or whoever.
> 
> >  plus in the US there is tariff.
> 
> Ah.  The US.  That explains a lot.

> 👤 **ikawrakow** replied on **2025-10-31** at **13:25:26**
> 
> > Also, anyone with some spare machines (I only have multi-GPU setups) can provide an SSH access etc. but I never seen any such requests from the IK or whoever.
> 
> Where do I post such a request?

> 👤 **Ph0rk0z** replied on **2025-10-31** at **13:50:27**
> 
> >What about the latency? You see, the tool from nvidia-toolkit I was mentioning, it provides such info. nvtop doesn't.
> 
> Latency is much improved when I did the P2P driver mod. I already tried all the simple stuff. People do literal 1x links on multi-gpu rigs. Until you do parallel computation PCIE speed isn't that massive of a deal. On mistral large this morning I get 368 t/s in 5bpw. I also tried the speculative decoding there and yea, it doesn't work either. Prompt processing above 100 is pretty tolerable as long as t/s above 10. I originally asked how it's tied to quants because of the big variance I saw playing with the REAP models. If x_KM is faster than IQ on both prompt and t/s then why am I downloading IQ quants, etc? But I don't have the internet to play with ones that count and the small models aren't representative because my rig just chews through them as gimpy as it is.
> 
> >Where do I post such a request?
> 
> I think you should have made a discussion. I bet there are lots of people with multi-gpu and multi-cpu machines that have nowhere to really go as most frameworks are either for exclusively GPU or home users with consumer machines. DDR5 prices are way way up there compared to DDR4.
> 
> if my pcm-memory look like that when decoding, I bet t/s would be way closer to GPU speeds
> 
> <img width="1035" height="286" alt="ik_cpu_max" src="https://github.com/user-attachments/assets/58c06caf-3424-44d4-9039-b84b23d8d7cb" />

> 👤 **magikRUKKOLA** replied on **2025-10-31** at **13:53:00**
> 
> @ikawrakow 
> > Where do I post such a request?
> 
> For a multi-GPU machine?  Well, you could generate the public key (ssh-keygen) and ask me to add it into ~/.ssh/authorized_keys.  The only inconvenience is that I do not have a dedicated IP address to access it via the clearnet.  Probably the easiest solution would be to generate an onion domain so you could access it via torsocks?  Alternatively, I can order some VPS with a dedicated IP address and build a tunnel to my machine so everything will be available via the clearnet IP.
> Which option would you prefer?