# MAIR: Multi-view Attention Inverse Rendering with 3D Spatially-Varying Lighting Estimation <br>

### [Project Page](https://bring728.github.io/mair.project/) | [Paper](https://arxiv.org/abs/2303.12368) | [Code(Test Only)](https://github.com/bring728/MAIR_Open) 


This is the official webpage for MAIR's OpenRooms FF dataset. 

_**For full access to the dataset, please fill out the form in [this file](https://github.com/bring728/OpenRooms_FF/blob/main/Request_for_Dataset.pdf) and send it to jhcho@kist.re.kr. We will then give you access to the dataset via your email.**_

For any questions, please email: happily@kist.re.kr.

## Dataset Introduction
OpenRooms FF(Forward Facing) is a dataset that extends [OpenRooms](https://github.com/ViLab-UCSD/OpenRooms#dataset-creation) into a multi-view setup. Each image set consists of 9 images looking in the same direction. For a detailed description of dataset creation, please refer to the paper's supplementary. 

## Dataset Overview
Since OpenRooms FF is based on images from OpenRooms, OpenRooms FF's scenes are also rendered in 6 versions: `main_xml`, `main_xml1`, `mainDiffMat_xml`, `mainDiffMat_xml1`, `mainDiffLight_xml` and `mainDiffLight_xml1`. The format of the file name is `<img_ind>_<data_type>_<view_ind>`.

`<img_ind>` indicates the index corresponding to the original openrooms image. For example, if the file name is `main_xml1/scene0001_00/8_*`, it means that it was reproduced from the image(camera pose) of `main_xml1/scene0001_00/*_8` in the original openrooms. `<data_type>` indicates the type of data (ex. im, imnormal, immask, etc.) and `<view_ind>` indicates the multi-view index (1-9) as shown in the image below. 

<img src = "https://github.com/bring728/OpenRooms_FF/blob/main/example1.png" width="640" height="240">

The training/testing split of the scenes can be found in [this link](https://drive.google.com/file/d/1HG93tgiShdizzUW80NY19DOLCUG_mAWS/view?usp=share_link). 

1. **[Dataset split](https://drive.google.com/file/d/1s_hAUwfD-Uz8yqlx3jULA_HX3_qSRTPb/view?usp=share_link)**: This is the Training / Test dataset split used in the MAIR experiment.
    However, since this split is not divided per scene, **we strongly recommend using this original OpenRooms [Training](https://drive.google.com/file/d/1ZhHaOPtnICGnEp9cgXT0HPaEm2SkdIJJ/view?usp=sharing) / [Test](https://drive.google.com/file/d/11HNfjg50Ku5ufEwiLzvRJ_SVzsk0eYU2/view?usp=sharing) dataset split.**


2. **[Image](https://drive.google.com/file/d/1s_hAUwfD-Uz8yqlx3jULA_HX3_qSRTPb/view?usp=share_link)**: The 480 × 640 HDR images `<img_ind>_im_<view_ind>.rgbe`, which can be read with the python command.
    ```python
    im = cv2.imread(imName, -1)[:, :, ::-1]
    ```

3. **[Material](https://drive.google.com/file/d/1q9OSU3QTSMujzu4uxebBpfzBAYYBir6s/view?usp=share_link)**: The 480 × 640 diffuse albedo maps `<img_ind>_imbaseColor_<view_ind>_.png` and roughness map `<img_ind>_imroughness_<view_ind>.png`. Note that the diffuse albedo map is saved in sRGB space. To load it into linear RGB space, we can use the following python commands. The roughness map is saved in linear space and can be read directly.
    ```python
    im = cv2.imread(imName)[:, :, ::-1]
    im = (im.astype(np.float32 ) / 255.0) ** (2.2)
    ```

4. **[Geometry](https://drive.google.com/file/d/1j1NIoYN56Zp5X39kK5QbFyaUQKh1BpTc/view?usp=share_link)**: The 480 × 640 normal maps `<img_ind>_imnomral_<view_ind>.png` and depth maps `<img_ind>_imdepth_<view_ind>.dat`. The R, G, B channel of the normal map corresponds to right, up, backward direction of the image plane. To load the depth map, we can use the following python commands.
    ```python
    with open(imName, 'rb') as fIn:
        # Read the height and width of depth
        hBuffer = fIn.read(4)
        height = struct.unpack('i', hBuffer)[0]
        wBuffer = fIn.read(4)
        width = struct.unpack('i', wBuffer)[0]
        # Read depth
        dBuffer = fIn.read(4 * width * height )
        depth = np.array(
            struct.unpack('f' * height * width, dBuffer ),
            dtype=np.float32 )
        depth = depth.reshape(height, width)
    ```


5. **[Predicted Depth](https://drive.google.com/file/d/1YEGHUwerrwmD_ZGSkVKSnW7gy99jkhcI/view?usp=share_link)**: We also provide predicted 480 × 640 depth maps `<img_ind>_cdsdepthest_<view_ind>.dat` and its confidence map `<img_ind>_cdsconf_<view_ind>.dat` obtained using [CDS-MVSNet](https://github.com/TruongKhang/cds-mvsnet) that we used in our experiments. The method of reading the file is the same as above.


6. **[Mask](https://drive.google.com/file/d/1kmBv8WL_ePnrbbtY0-1NnvLI6LhHXsCv/view?usp=share_link)**: The 480 × 640 grey scale mask `<img_ind>_immask_<view_ind>.png` for light sources. The pixel value 0 represents the region of environment maps. The pixel value 0.5 represents the region of lamps. Otherwise, the pixel value will be 1. 


7. **[SVLighting](https://drive.google.com/file/d/15xg7o0b_7M1o0-_vLk1iQJ3ROIqfk617/view?usp=share_link)**: The (120 × 8) × (160 × 16) per-pixel environment maps `<img_ind>_imenvlow_<view_ind>.hdr`. The spatial resolution is 120 x 160 while the environment map resolution is 8 x 16. To read the per-pixel environment maps, we can use the following python commands.
    ```python
    # Read the envmap of resolution 960 x 2560 x 3 in RGB format
    env = cv2.imread(imName, -1)[:, :, ::-1]
    # Reshape and permute the per-pixel environment maps
    env = env.reshape(120, 8, 160, 16, 3)
    env = env.transpose(0, 2, 1, 3, 4)
    ```
    We recommend using [Rclone](https://rclone.org/) to avoid slow or unstable downloads.

8. **[SVLightingDirect](https://drive.google.com/file/d/1sTsyG63bo0Kn0YebYG_U8niyQ8Jskzw_/view?usp=share_link)**: The (30 × 16) × (40 × 32) per-pixel environment maps with direct illumination only `<img_ind>_imenvDirect_<view_ind>.hdr`. The spatial resolution is 30 × 40 while the environment maps resolution is 16 × 32. The direct per-pixel environment maps can be load the same way as the per-pixel environment maps. 


9. **[Camera](https://drive.google.com/file/d/1Z1TN71qJ26zypxcrQn5pweaFRe2xu1C3/view?usp=share_link)**: The 3 x 6 x 9 Camera intrinsic, extrinsic parameters and scene boundary `<img_ind>_cam_mats.npy`.
    ```python
    # Read all cameras (3 x 6 x 9).
    cam_mats = np.load(camName)
    # Read camera 5 (<view_ind> is 5)
    cam_5 = cam_mats[:, :, 4]
    
    # Read camera 1 (<view_ind> is 1)
    cam_1 = cam_mats[:, :, 0]
    
    # read camera to world matrix (3 x 4)
    # camera coordinates axes are right, down, forward. 
    camera_to_world = cam_1[:, :4]
    
    # image height, width, focal length
    h, w, f = cam_1[:, 4]
    
    # minimum depth, maximum depth.
    min_z, max_z, _ = cam_1[:, 5]
    ```

## Related Datasets
The OpenRooms FF dataset is built on several prior works, as noted below.
1. **[OpenRooms dataset](https://github.com/ViLab-UCSD/OpenRooms)**: The original OpenRooms dataset.
1. **[ScanNet dataset](http://www.scan-net.org/)**: The real 3D scans of indoor scenes.
1. **[Scan2cad dataset](https://github.com/skanti/Scan2CAD)**: The alignment of CAD models to the scanned point clouds.
1. **[Laval outdoor lighting dataset](http://outdoor.hdrdb.com/)**: HDR outdoor environment maps
1. **[HDRI Haven lighting dataset](https://hdrihaven.com/)**: HDR outdoor environment maps
1. **[PartNet dataset](https://partnet.cs.stanford.edu/)**: CAD models
2. **[Adobe Stock](https://stock.adobe.com/search?filters%5Bcontent_type%3A3d%5D=1&filters%5B3d_type_id%5D%5B0%5D=3&load_type=3d+lp)**: High-quality microfacet SVBRDF texture maps. Please license the materials from Adobe Stock.



## Citation
If you find our work is useful, please consider cite:
```
@article{choi2023mair,
  title={MAIR: Multi-view Attention Inverse Rendering with 3D Spatially-Varying Lighting Estimation},
  author={Choi, JunYong and Lee, SeokYeong and Park, Haesol and Jung, Seung-Won and Kim, Ig-Jae and Cho, Junghyun},
  journal={arXiv preprint arXiv:2303.12368},
  year={2023}
}

```

