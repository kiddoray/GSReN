### Environment

项目路径

```
cd /data/feiben/gaussian_splatting
```

Activate conda env

```
conda activate gaussian_splatting
```

### Dataset preparation

#### Create your own datasets

##### 1. 准备想要重建的场景的多视角照片

可以拍摄视频，然后使用 ffmpeg 提取照片

```
FFMPEG -i {path to video} -qscale:v 1 -qmin 1 -vf fps={frame extraction rate} %04d.jpg
```

然后将所获得的照片放置于项目文件下的 `data/{scene name}/input/` 中

##### 2. 使用 COLMAP 从照片中提取相机位姿和稀疏点云

在项目文件夹下运行以下命令

```
python convert.py -s <data/{scene name}> [--no_gpu] #If not use a CUDA-powered COLMAP
```

成功运行结束后将得到以下目录结构：

<img src="https://cdn.jsdelivr.net/gh/olallaland/typora-image-cloud@main/images/2023/11/03/20231103135333.png" alt="image-20231102152803005" style="zoom:50%;" />

<img src="https://cdn.jsdelivr.net/gh/olallaland/typora-image-cloud@main/images/2023/11/03/20231103135333-1.png" alt="image-20231102153357788" style="zoom:50%;" />

**可视化重建的相机位姿和稀疏点云（非必需）**

打开 COLMAP GUI，依次选择 File -> Import Model，在弹出窗口中选择对应目录下的 `sparse/0` 文件夹，即可得到可视化结果

<img src="https://cdn.jsdelivr.net/gh/olallaland/typora-image-cloud@main/images/2023/11/03/20231103135333-3.png" alt="image-20231102154109274" style="zoom:50%;" />

#### Use other datasets

下载对应的数据集，将其放在 data 目录下即可

<img src="https://cdn.jsdelivr.net/gh/olallaland/typora-image-cloud@main/images/2023/12/21/20231221094837.png" alt="image-20231221093820942" style="zoom:50%;" />

> Mip-NeRF360 dataset http://storage.googleapis.com/gresearch/refraw360/360_v2.zip
>
> NeRF-Synthetic dataset https://drive.google.com/drive/folders/1KHhljnqLvIJkRkaqQ8TaeBZirMsnDAhf

### Training

在项目文件夹下，运行以下命令，即可开始训练

```
python train.py -s <path to COLMAP or NeRF Synthetic dataset, e.g. "data/bottle2"> --eval -m <optional, a random name will be generated if not specified> 
```

训练结果会存储在 log 中的 "Output folder" 下

![image-20231102153225484](https://cdn.jsdelivr.net/gh/olallaland/typora-image-cloud@main/images/2023/11/03/20231103135333-2.png)

<img src="https://cdn.jsdelivr.net/gh/olallaland/typora-image-cloud@main/images/2023/11/03/20231103135333-4.png" alt="image-20231102154134223" style="zoom:50%;" />

在包含 depth prior loss 计算的分支中，训练时可通过 `--depth_prediction_model`  指定使用的预训练的单目深度图像预测模型，目前共有两种模型可选：”ZoeD_NK“ 和 ”DPT“，默认使用 ”ZoeD_NK“

### Rendering

rendering 是 evaluation 的前置步骤，如果要做 evaluation 的话，需要先完成在 test split 上的 rendering

在项目文件夹下，运行以下命令，即可开始渲染

```
python render.py -s <path to COLMAP or NeRF Synthetic dataset, e.g. "data/bottle2"> -m <path to trained model, e.g. output/50925394-2> --eval 
```

在 exp_add_video_rendering 以及 cp 了 "add video rendering" 这个 commit 的分支下，运行下列命令开始渲染图像：

```
python render.py -s <path to COLMAP or NeRF Synthetic dataset, e.g. "data/bottle2"> -m <path to trained model, e.g. output/50925394-2> --eval --render_images
```

运行下列命令开始渲染视频：

```
python render.py -s <path to COLMAP or NeRF Synthetic dataset, e.g. "data/bottle2"> -m <path to trained model, e.g. output/50925394-2> --eval --render_video --fps <setting the fps of rendered video> --camera_type <"ellipse" or "spiral">
```

### Evaluation

在项目文件夹下，运行以下命令

```
python metrics.py -m <path to trained model, e.g. output/50925394-2>
```

即可得到模型在测试集上的 SSIM、PSNR 以及 LPIPIS 结果

<img src="https://cdn.jsdelivr.net/gh/olallaland/typora-image-cloud@main/images/2023/12/21/20231221094827.png" alt="image-20231221094820806" style="zoom:50%;" />