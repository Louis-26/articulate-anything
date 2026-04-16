# Device prepare
GPU device: A100
cuda version: 12.8

# step 1: set up the environment
enable an interactive session on GPU cluster first, then
```bash    
conda create -n articulate-anything python=3.9 -y
conda activate articulate-anything
cd $(git rev-parse --show-toplevel)
pip install -e .

pip install hydra-core --upgrade
# certify by
python -c "import hydra; print(hydra.__version__)"

# adjust version
pip install huggingface_hub==0.23.5
```
# step 2: prepare dataset
```bash
cd $(git rev-parse --show-toplevel)
pip install huggingface_hub
pip install -U "huggingface_hub[cli]"
if [[ ":$PATH:" != *":$CONDA_PREFIX/bin:"* ]]; then
    export PATH="$CONDA_PREFIX/bin:$PATH"
fi
hf --help > /dev/null 2>&1; if [ $? -eq 0 ]; then echo "pass"; else echo "huggingface-cli not found"; fi
hf download vlongle/articulate-anything-dataset-preprocessed --repo-type dataset --local-dir ./datasets 

cd datasets
mv partnet-mobility-v0-processed.zip partnet-mobility-v0.zip
mkdir partnet-mobility-v0
# this will generate 499498 files in total, make sure the server can support that
UNZIP_DISABLE_ZIPBOMB_DETECTION=TRUE unzip partnet-mobility-v0.zip 
```


if file number is strictly limited, just select 100 objects out of 2347 objects to unzip, otherwise it will exceed the disk quota 
```bash
unzip -Z1 partnet-mobility-v0.zip 'partnet-mobility-v0/dataset/*/' \
  | awk -F/ '{print $3}' | awk '!seen[$0]++ && NF' | head -n 100 \
  | xargs -I{} unzip -q partnet-mobility-v0.zip "partnet-mobility-v0/dataset/{}/*"

# rm -rf partnet-mobility-v0.zip 
# cd .. && rm -rf articulate-anything-dataset-preprocessed
```


# step 3: preprocess

```bash
conda activate articulate-anything
cd "$(git rev-parse --show-toplevel)/articulate_anything/preprocess"

# this is what I use as parameter, we can finetune that

python preprocess_partnet.py parallel=4 modality=text obj_ids=693 dataset_dir=../../datasets/partnet-mobility-v0/dataset 
```

# step 4: demo
```bash
cd $(git rev-parse --show-toplevel)
python gradio_app.py # ❌ not working for some reason
```


# Experiment
download cotracker2.pth
```bash
cd "$(git rev-parse --show-toplevel)/articulate_anything"
mkdir -p co-tracker/checkpoints
cd co-tracker/checkpoints
wget https://huggingface.co/facebook/cotracker/resolve/main/cotracker2.pth
```

change relative path, under `conf/cotracker/default.yaml`
```yaml
# original
checkpoint_path: "../co-tracker/checkpoints/cotracker2.pth"
# change to
checkpoint_path: "articulate_anything/co-tracker/checkpoints/cotracker2.pth"
```

## ✅💾 PartNet-Mobility Masked Reconstruction
```bash
python articulate.py modality=partnet prompt=693 out_dir=results additional_prompt=joint_0
```

## ✅🖋 Text Articulation

```bash
cd $(git rev-parse --show-toplevel)
# preprocess
python articulate_anything/preprocess/preprocess_partnet.py parallel=4 modality=text
pip install openai==1.30.5

# articulate

python articulate.py modality=text prompt="suitcase with a retractable handle" out_dir=results/text/suitcase joint_actor.mode=text joint_actor.targetted_affordance=false

```

## 🖼 / 🎥 Visual Articulation
```bash
# preprocess
python articulate_anything/preprocess/preprocess_partnet.py parallel=4 modality=image

```