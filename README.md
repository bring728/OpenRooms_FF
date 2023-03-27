# MAIR: Multi-view Attention Inverse Rendering with 3D Spatially-Varying Lighting Estimation <br>


This is the official webpage for [MAIR](https://bring728.github.io/mair.project/)'s OpenRooms FF dataset


## Dataset Introduction
OpenRooms FF(Forward Facing) is a dataset that extends [OpenRooms](https://github.com/ViLab-UCSD/OpenRooms#dataset-creation) into a multi-view setup. Each image set consists of 9 images looking in the same direction. For a detailed description of dataset creation, please refer to the paper's supplementary. For any questions, please email: happily@kist.re.kr

## Dataset Overview
In the following, we explain each data details and how to read it. Since OpenRooms FF is based on images from OpenRooms, OpenRooms FF's scenes are also rendered in 6 versions: `main_xml`, `main_xml1`, `mainDiffMat_xml`, `mainDiffMat_xml1`, `mainDiffLight_xml` and `mainDiffLight_xml1`. The format of the file name is <img_ind>_<data_type>_<view_ind>.<file_type>. 
<img_ind> indicates the index corresponding to the original openrooms image, <data_type> indicates the type of data (ex. im, imnormal, immask, etc.), <view_ind> indicates the multi-view index (1-9) as shown in the image below, and <file_type> indicates the file extension (png, hdr, etc.). For example, if the file name is main_xml1/scene0001_00/8_imnormal_3.png, it means that it was reproduced from main_xml1/scene0001_00/im*_8 of the original openrooms.

<img src = "https://github.com/bring728/OpenRooms_FF/blob/main/example1.png" width="640" height="240">

The training/testing split of the scenes can be found in [scene split](https://drive.google.com/file/d/1HG93tgiShdizzUW80NY19DOLCUG_mAWS/view?usp=share_link). 

1. **[Image](https://drive.google.com/file/d/1s_hAUwfD-Uz8yqlx3jULA_HX3_qSRTPb/view?usp=share_link)**: The 480 × 640 HDR images `<img_ind>_im_<view_ind>.hdr`, which can be read with the python command.
    ```python
    im = np.array(imageio.imread_v2(imName, format='HDR-FI'))
    ```

2. **[Material](https://drive.google.com/file/d/1q9OSU3QTSMujzu4uxebBpfzBAYYBir6s/view?usp=share_link)**: The 480 × 640 diffuse albedo maps `<img_ind>_imbaseColor_<view_ind>_.png` and roughness map `<img_ind>_imroughness_<view_ind>.png`. Note that the diffuse albedo map is saved in sRGB space. To load it into linear RGB space, we can use the following python commands. The roughness map is saved in linear space and can be read directly.
    ```python
    im = cv2.imread(imName)[:, :, ::-1]
    im = (im.astype(np.float32 ) / 255.0) ** (2.2)
    ```

3. **[Geometry](https://drive.google.com/file/d/1j1NIoYN56Zp5X39kK5QbFyaUQKh1BpTc/view?usp=share_link)**: The 480 × 640 normal maps `<img_ind>_imnomral_<view_ind>.png` and depth maps `<img_ind>_imdepth_<view_ind>.dat`. The R, G, B channel of the normal map corresponds to right, up, backward direction of the image plane. To load the depth map, we can use the following python commands.
    ```python
    with open('imdepth_1.dat', 'rb') as fIn:
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

4. **[Predicted Depth](https://drive.google.com/file/d/1YEGHUwerrwmD_ZGSkVKSnW7gy99jkhcI/view?usp=share_link)**: We also provide SOTA MVS[CDS-MVSNet](https://github.com/TruongKhang/cds-mvsnet) depth maps `<img_ind>_imdepth_<view_ind>.dat` and confidence map that we used in our experiments. The method of reading the file is the same as above.


5. **[Mask](https://drive.google.com/file/d/1kmBv8WL_ePnrbbtY0-1NnvLI6LhHXsCv/view?usp=share_link)**: The 480 × 460 grey scale mask `immask_*.png` for light sources. The pixel value 0 represents the region of environment maps. The pixel value 0.5 represents the region of lamps. Otherwise, the pixel value will be 1. 


6. **[SVLighting]()**: The (120 × 8) × (160 × 16) per-pixel environment maps `imenv_*.hdr`. The spatial resolution is 120 x 160 while the environment map resolution is 16 x 32. To read the per-pixel environment maps, we can use the following python commands.
    ```python
    # Read the envmap of resolution 1920 x 5120 x 3 in RGB format
    env = cv2.imread('imenv_1', -1)[:, :, ::-1]
    # Reshape and permute the per-pixel environment maps
    env = env.reshape(120, 16, 160, 32, 3)
    env = env.transpose(0, 2, 1, 3, 4)
    ```
    We render per-pixel environment maps for `main_xml(1)`, `mainDiffMat_xml(1)` and `mainDiffLight_xml(1)`. Since the total size of per-pixel environment maps is 4.0 TB, we do not provide an extra .zip format for downloading. Please consider using the tool [Rclone](https://rclone.org/) if you hope to download all the per-pixel environment maps.

8. **[SVLightingDirect]()**: The (30 × 16) × (40 × 32) per-pixel environment maps with direct illumination `imenvDirect_*.hdr` only. The spatial resolution is 30 × 40 while the environment maps resolution is 16 × 32. The direct per-pixel environment maps can be load the same way as the per-pixel environment maps. We only render direct per-pixel environment maps for `main_xml(1)` and `mainDiffLight_xml(1)` because the direct illumination of `mainDiffMat_xml(1)` is the same as `main_xml(1)`.

