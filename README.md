# Update: v1.0.2 even superfaster mode edition

## THE GUI IS OUT <br>
See [GUI Usage tutorial for dummies](GUI_TUTORIAL.md)

## The hlky's webui version [is out](https://github.com/neonsecret/stable-diffusion-webui)

## The superfast and low-vram mode have been updated, speed_mp parameter introduced.
Below you can see the comparison table.
<br>
| resolution 	| steps 	| speed_mp 	| time          	| vram 	| low vram mode 	|
|------------	|-------	|----------	|---------------	|------	|---------------	|
| 512x512    	| 50    	| 1        	| 1.5 minutes   	| 4    	| no            	|
| 512x512    	| 50    	| 1        	| 36 seconds    	| 8    	| no            	|
| 512x512    	| 50    	| 1        	| 18 seconds    	| 24   	| no            	|
| 1024x1024  	| 30    	| 1        	| 10 minutes    	| 4    	| yes           	|
| 1024x1024  	| 50    	| 3        	| 15 minutes    	| 4    	| no            	|
| 1024x1024  	| 50    	| 1        	| 2.306 minutes 	| 8    	| no            	|
| 1024x1024  	| 50    	| 1        	| 70 seconds    	| 24   	| no            	|
| 1600x1600  	| 50    	| 3        	| 12.6 minutes  	| 8    	| no            	|
| 1600x1600  	| 50    	| 3        	| 4.5 minutes   	| 24   	| no            	|
| 1600x1600  	| 50    	| 2        	| 4.3 minutes   	| 24   	| no            	|
| 1984x1984  	| 30    	| 3        	| 7.5 minutes   	| 24   	| no            	|
| 2048x2048  	| 30    	| 3        	| 8.659 minutes 	| 24   	| no            	|
| 2048x2048  	| 50    	| 3        	| 13.44 minutes 	| 24   	| no            	|
| 2048x2048  	| 31    	| 11       	| 1 hour        	| 24   	| yes           	|
| 512x4096   	| 50    	| 2        	| 3 minutes     	| 24   	| no            	|
| 3840x2112  	| 1     	| 11       	| 25 minutes    	| 24   	| yes           	|
<br>

gpus used: gtx 1050 ti, rtx 3070, rtx 3090
(huge thanks to @therustysmear for helping me in these tests)

speed_mp parameter basically allows to generate bigger-resolution images at the cost of speed, values higher then 10 are possible but not recommended.

### How to generate so high-res images?
The superfast mode is enabled by default, however if you encounter OOM errors or want to go higher in resolution, disable it:

Example cli command with txt2img and high-res mode:
```
python optimizedSD/optimized_txt2img.py --prompt "an apple" --config_path optimizedSD/v1-inference_lowvram.yaml --H 512 --W 512 --seed 27 --n_iter 1 --n_samples 1 --ddim_steps 50
```
Example gradio command with txt2img and high-res mode:
```
python optimizedSD/txt2img_gradio.py --config_path optimizedSD/v1-inference_lowvram.yaml
```
the `--config_path optimizedSD/v1-inference_lowvram.yaml` argument enables a low-vram mode which allows to generate bigger-resolution images at the slight cost of the speed.

<h1 align="center">Optimized Stable Diffusion</h1>
<p align="center">
    <img src="https://img.shields.io/github/last-commit/neonsecret/stable-diffusion?logo=Python&logoColor=green&style=for-the-badge"/>
        <img src="https://img.shields.io/github/issues/neonsecret/stable-diffusion?logo=GitHub&style=for-the-badge"/>
                <img src="https://img.shields.io/github/stars/neonsecret/stable-diffusion?logo=GitHub&style=for-the-badge"/>
    <a href="https://colab.research.google.com/github/neonsecret/stable-diffusion/blob/main/optimized_colab.ipynb">
      <img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab"/>
    </a>
</p>

This repo is a modified version of the Stable Diffusion repo, optimized to use less VRAM than the original by
sacrificing inference speed.

To achieve this, the stable diffusion model is fragmented into four parts which are sent to the GPU only when needed.
After the calculation is done, they are moved back to the CPU. This allows us to run a bigger model while requiring less
VRAM.

Also I invented the sliced atttention technique, which allows to push the model's abilities even further. It works by automatically determining the slice size from your vram and image size and then allocating it one by one accordingly. 

<h1 align="center">Installation</h1>


You can clone this repo and follow the same installation steps as the original (mainly creating the conda environment and
placing the weights at the specified location). <br>
So run: <br>
`conda env create -f environment.yaml` <br>
`conda activate ldm`

<h2 align="center">Additional steps for AMD Cards</h2>

After activating your conda environment, you have to update torch and torchvision wheels which were built with ROCm support (only on linux):

`pip3 install --upgrade torch torchvision --extra-index-url https://download.pytorch.org/whl/rocm5.1.1`

<h2 align="center">Docker</h2>

Alternatively, if you prefer to use Docker, you can do the following:

1. Install [Docker](https://docs.docker.com/engine/install/)
   , [Docker Compose plugin](https://docs.docker.com/compose/install/),
   and [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html#docker)
2. Clone this repo to, e.g., `~/stable-diffusion`
3. Put your downloaded `model.ckpt` file into `~/sd-data` (it's a relative path, you can change it
   in `docker-compose.yml`)
4. `cd` into `~/stable-diffusion` and execute `docker compose up --build`

This will launch gradio on port 7860 with txt2img. You can also use `docker compose run` to execute other Python
scripts.

<h1 align="center">Usage</h1>

## img2img

- `img2img` can generate _512x512 images from a prior image and prompt on a 4GB VRAM GPU in under 20 seconds per image_
  on an RTX 2060.

- The maximum size that can fit on 6GB GPU (RTX 2060) is around 576x768.

- For example, the following command will generate 20 512x512 images:

`python optimizedSD/optimized_img2img.py --prompt "Austrian alps" --init-img ~/sketch-mountains-input.jpg --strength 0.8 --n_iter 2 --n_samples 10 --H 512 --W 512`

## txt2img

- `txt2img` can generate _512x512 images from a prompt on a 4GB VRAM GPU in under 25 seconds per image_ on an RTX 2060.

- For example, the following command will generate 20 512x512 images:

`python optimizedSD/optimized_txt2img.py --prompt "Cyberpunk style image of a Telsa car reflection in rain" --H 512 --W 512 --seed 27 --n_iter 2 --n_samples 10 --ddim_steps 50`

## inpainting

- `inpaint_gradio.py` can fill masked parts of an image based on a given prompt. It can inpaint 512x512 images while
  using under 4GB of VRAM.

- To launch the gradio interface for inpainting, run `python optimizedSD/inpaint_gradio.py`. The mask for the image can
  be drawn on the selected image using the brush tool.

- The results are not yet perfect but can be improved by using a combination of prompt weighting, prompt engineering and
  testing out multiple values of the `--strength` argument.

- _Suggestions to improve the inpainting algorithm are most welcome_.

## img2img interpolation

- `img2img_interpolate.py` creates an animation of image transformation using a text prompt

- To launch the gradio interface for inpainting, run `python optimizedSD/img2img_interpolate.py`. The mask for the image can
  be drawn on the selected image using the brush tool.

- The results are not yet perfect but can be improved by using a combination of prompt weighting, prompt engineering and
  testing out multiple values of the `--strength` argument.


<h1 align="center">Using the Gradio GUI</h1>

- You can also use the built-in gradio interface for `img2img`, `txt2img` & `inpainting` instead of the command line
  interface. Activate the conda environment and install the latest version of gradio using `pip install gradio`,

- Run img2img using `python optimizedSD/img2img_gradio.py`, txt2img using `python optimizedSD/txt2img_gradio.py` and
  inpainting using `python optimizedSD/inpaint_gradio.py`.

- img2img_gradio.py has a feature to crop input images. Look for the pen symbol in the image box after selecting the
  image.

<h1 align="center">Arguments</h1>

## `--seed`

**Seed for image generation**, can be used to reproduce previously generated images. Defaults to a random seed if
unspecified.

- The code will give the seed number along with each generated image. To generate the same image again, just specify the
  seed using `--seed` argument. Images are saved with its seed number as its name by default.

- For example if the seed number for an image is `1234` and it's the 55th image in the folder, the image name will be
  named `seed_1234_00055.png`.

## `--n_samples`

**Batch size/amount of images to generate at once.**

- To get the lowest inference time per image, use the maximum batch size `--n_samples` that can fit on the GPU.
  Inference time per image will reduce on increasing the batch size, but the required VRAM will increase.

- If you get a CUDA out of memory error, try reducing the batch size `--n_samples`. If it doesn't work, the other option
  is to reduce the image width `--W` or height `--H` or both.

## `--n_iter`

**Run _x_ amount of times**

- Equivalent to running the script n_iter number of times. Only difference is that the model is loaded only once per
  n_iter iterations. Unlike `n_samples`, reducing it doesn't have an effect on VRAM required or inference time.

## `--H` & `--W`

**Height & width of the generated image.**

- Both height and width should be a multiple of 64.

## `--turbo`

**Increases inference speed at the cost of extra VRAM usage.**

- Using this argument increases the inference speed by using around 1GB of extra GPU VRAM. It is especially effective
  when generating a small batch of images (~ 1 to 4) images. It takes under 25 seconds for txt2img and 15 seconds for
  img2img (on an RTX 2060, excluding the time to load the model). Use it on larger batch sizes if GPU VRAM available.

## `--precision autocast` or `--precision full`

**Whether to use `full` or `mixed` precision**

- Mixed Precision is enabled by default. If you don't have a GPU with tensor cores (any GTX 10 series card), you may not
  be able use mixed precision. Use the `--precision full` argument to disable it.

## `--format png` or `--format jpg`

**Output image format**

- The default output format is `png`. While `png` is lossless, it takes up a lot of space (unless large portions of the
  image happen to be a single colour). Use lossy `jpg` to get smaller image file sizes.

## `--unet_bs`

**Batch size for the unet model**

- Takes up a lot of extra RAM for **very little improvement** in inference time. `unet_bs` > 1 is not recommended!

- Should generally be a multiple of 2x(n_samples)

<h1 align="center">Weighted Prompts</h1>

- Prompts can also be weighted to put relative emphasis on certain words.
  eg. `--prompt tabby cat:0.25 white duck:0.75 hybrid`.

- The number followed by the colon represents the weight given to the words before the colon. The weights can be both
  fractions or integers.

## Changelog

- v0.8: Added gradio interface for inpainting.
- v0.7: Added support for logging, jpg file format
- v0.6: Added support for using weighted prompts. (based on
  @lstein's [repo](https://github.com/lstein/stable-diffusion))
- v0.5: Added support for using gradio interface.
- v0.4: Added support for specifying image seed.
- v0.3: Added support for using mixed precision.
- v0.2: Added support for generating images in batches.
- v0.1: Split the model into multiple parts to run it on lower VRAM.
