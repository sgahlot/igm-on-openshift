# Stable Diffusion XL Runtime

A custom runtime to deploy the [Stable Diffusion XL](https://huggingface.co/stabilityai/stable-diffusion-xl-base-1.0) model (or models from the same family) using [KServe](https://kserve.github.io/website/latest/) and [Diffusers](https://huggingface.co/docs/diffusers/index).

## Server - Container image

The folder `kserve-sdxl-container` contains the server code, as well as the Containerfile to build a containerized image.

It is based on the kserve and the diffusion frameworks, and supports using the base SDXL image only, or base+refiner, **_with already generated LoRA weights_**. The models are loaded as `fp16`, from `safetensors` files.


### Handler

Python file `server.py` is modified for the following:
* Load LoRA weights that are specified by `--lora_dir` flag
  * The LoRA weights are loaded to support the model that is fine-tuned
* More debug messages to show what's happening with model loading and inference call


### Build the image (optional)

To build the image, using the Containerfile, use the following commands:
```
podman build --platform=linux/amd64 -t <IMAGE_NAME>:<VERSION>

e.g. podman build --platform=linux/amd64 -t quay.io/sgahlot/stable-diffusion-igm:1.0
```

Once the image is successfully built, use the following command to push the image to the registry:
```
podman push <IMAGE_NAME>:<VERSION>

e.g. podman push quay.io/sgahlot/stable-diffusion-igm:1.0
```


**NOTE**: _The following image can be used in custom ServingRuntime: `quay.io/sgahlot/stable-diffusion-igm:1.0` if one does not want to build the image._


### Parameters

The parameters you can pass as arguments to the script (or container image) are:

- All the standard parameters from kserve [model_server](https://github.com/kserve/kserve/blob/master/python/kserve/kserve/model_server.py).
- `--model_id`: Model ID to load. Defaults to `/mnt/models`. You must adapt this if you use the refiner model and need to point to a specific directory in your models folder.
- `--lora_dir`: LoRA weights directory
- `--single_file_model`: Full name/location of your model if saved as a single file.
- `--use_refiner`: True/False (default False) indicates if the refiner must be used.
- `--refiner_id`: Refiner model ID to load. You must adapt this to point to a specific directory in your models folder.
- `--refiner_single_file_model`: Full name/location of your refiner model if saved as a single file.
- `--device`: Device to use, including offloading configuration. The values can be:
  - `cuda`: load all models (base+refiner) on the GPU
  - `enable_model_cpu_offload`: Full-model offloading, uses less GPU memory without much impact on inference.
  - `enable_sequential_cpu_offload`: Sequential CPU offloading preserves a lot of memory but it makes inference slower because submodules are moved to GPU as needed, and they’re immediately returned to the CPU when a new module runs.
  - `cpu`: only uses CPU and standard RAM. Available for tests/compatibility purposes, unusable in practice (way too slow...).

### Examples

The folder `kserve-sdxl-container` contains two example files on how to launch the server:

- `start-base.sh`: starts the server with the base model only, saved as a single file. CPU offloading is set to `enable_model_cpu_offload` to allow running on 8GB of VRAM.
- `start-refiner.sh`: starts the server with base+refiner models, both saved as singles files. CPU offloading is set to `enable_sequential_cpu_offload` to allow running on  8GB of VRAM (much less consumed, with longer inference time).

## Clients examples

Examples to use the inference point either with the base model only or the base+refiner are available in the notebook [kserve-sdxl-client-examples.ipynb](./kserve-sdxl-client-examples.ipynb).
