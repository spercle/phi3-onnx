Fine Tune Phi Model with MLX and export to Onnx 
=============

Tested on `Apple M1`


# Install HomeBrew 
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

# Setup Python Virtual Enviroment (venv)
```bash

python3.11 -m venv venv

source venv/bin/activate

# deactivate
```


# Setup MLX
``` bash

pip install --upgrade pip

pip install mlx mlx-lm 

## verify mlx version 

python -c "from importlib.metadata import version; print(version('mlx'))"


pip install torch transformers "optimum[exporters]" onnx
```


# Train
```bash
mlx_lm.lora --model microsoft/Phi-3-mini-4k-instruct --train --data ./data --iters 1000 
```

### Test Post Training 
```bash
#trained
mlx_lm.generate --model microsoft/Phi-3-mini-4k-instruct --adapter-path ./adapters --max-token 2048 --prompt "Why do chameleons change colors? "

#orignial
mlx_lm.generate --model microsoft/Phi-3-mini-4k-instruct --max-token 2048 --prompt "Why do chameleons change colors?"

```

# Fuse
```bash
mlx_lm.fuse --model "microsoft/Phi-3-mini-4k-instruct" --adapter-path "adapters/" --save-path "fused_model/"
```

### Test Fused 
```bash

mlx_lm.generate --model fused_model --max-token 2048 --prompt "Why do chameleons change colors? "

```

Epected Answer example (a Short Answer)
```python
==========
Chameleons change colors to communicate <|end|>
==========
```


# Quantizied
mlx `0.25.2`
Disable Card Creation
nano /Volumes/1TB/phi3-onnx/venv/lib/python3.11/site-packages/mlx_lm/convert.py
function (around line 162) and locate the line:
```python
# create_model_card(mlx_path, hf_path)
```


```bash
mlx_lm.convert --hf-path fused_model -q
```

### Test Quantzied Model

```bash
mlx_lm.generate --model mlx_model --max-token 2048 --prompt "Why do chameleons change colors? "

```

Epected Answer
```python
==========
Chameleons change colors to communicate <|end|>
==========
```


# Convert (14GB)
```bash
optimum-cli export onnx \
  --model ./fused_model \
  --task text-generation \
  ./onnx_model
```



optimum-cli export onnx \
  --model ./mlx_model \
  --task text-generation \
  --optimize 04 \ 
  ./onnx_model






## Linux Specific 

NOTE: MLX is Apple Only 

## TODOs 

[ ] or onnx_model,  use onnx runtime to test, if doesent work then process to others ( BEN )

[ ] test the fused model. (Model is in HF format i .e safetensors)  
  - pytourch?

[ ]  repeat for mlx_model 


[ ] deployable chatbot app 





## Example of files we need for chatbot app 

```
config.json
onnx/model_q4f16.onnx
onnx/model_q4f16.onnx_data
tokenizer.json
tokenizer_config.json
```
