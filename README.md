# MAIR: Multi-view Attention Inverse Rendering with 3D Spatially-Varying Lighting Estimation <br>


This is the official webpage for [MAIR](https://bring728.github.io/mair.project/)'s OpenRooms FF dataset


## Dataset Introduction
OpenRooms FF(Forward Facing) is a dataset that extends [OpenRooms](https://github.com/ViLab-UCSD/OpenRooms#dataset-creation) into a multi-view setup. Each image set consists of 9 images looking in the same direction. For a detailed description of dataset creation, please refer to the paper's supplementary. For any questions, please email: happily@kist.re.kr

## Dataset Overview
Since OpenRooms FF is based on images from OpenRooms, OpenRooms FF's scenes are also rendered in 6 versions: `main_xml`, `main_xml1`, `mainDiffMat_xml`, `mainDiffMat_xml1`, `mainDiffLight_xml` and `mainDiffLight_xml1``

In the following, we explain each data type and how to read it. The name of the file is <openrooms_image_index>_<data_type>_<view_index>.<file_type> openrooms_image_index indicates the index corresponding to the original openrooms image. data_type indicates the type of data (ex. im, imnormal, immask, etc.) and file_type indicates the file extension (png, hdr, etc.). view_index represents the multi-view index, in our case it is 1-9. 

<img src = "https://github.com/bring728/OpenRooms_FF/blob/main/example1.png" width="640" height="240">




The training/testing split of the scenes can be found in [scene split](https://drive.google.com/file/d/1HG93tgiShdizzUW80NY19DOLCUG_mAWS/view?usp=share_link). 

1. **[Image](https://drive.google.com/file/d/1s_hAUwfD-Uz8yqlx3jULA_HX3_qSRTPb/view?usp=share_link)**: The 480 × 640 HDR images `im_*.hdr`, which can be read with the python command.
    ```python
    im = cv2.imread('im_1.hdr', -1)[:, :, ::-1]
    ```
    We render images for `main_xml(1)`, `mainDiffMat_xml(1)` and `mainDiffLight_xml(1)`.

2. **[Material](https://drive.google.com/drive/folders/1E8wMJHDqnN7rArQH28_alTe870078Qpp?usp=sharing)** and **[Material.zip](https://drive.google.com/file/d/1yPFzSMxStjZtPN2vC43-p45Gq9NGFNPF/view?usp=sharing)**: The 480 × 640 diffuse albedo maps `imbaseColor_*.png` and roughness map `imroughness_*.png`. Note that the diffuse albedo map is saved in sRGB space. To load it into linear RGB space, we can use the following python commands. The roughness map is saved in linear space and can be read directly.
    ```python
    im = cv2.imread('imbaseColor_1.hdr')[:, :, ::-1]
    im = (im.astype(np.float32 ) / 255.0) ** (2.2)
    ```
    We only render the diffuse albedo maps and roughness maps for `main_xml(1)` and `mainDiffMat_xml(1)` because `mainDiffLight_xml(1)` share the same material maps with the `main_xml(1)`.

3. **[Geometry](https://drive.google.com/drive/folders/1i7IMlGf_LDjuHbM6sutwOO5LjzXcbVN2?usp=sharing)** and **[Geometry.zip](https://drive.google.com/file/d/1FRPMKIDzFN1puk27MMGaIxCoi6HmLGyE/view?usp=sharing)**: The 480 × 640 normal maps `imnomral_*.png` and depth maps `imdepth_*.dat`. The R, G, B channel of the normal map corresponds to right, up, backward direction of the image plane. To load the depth map, we can use the following python commands.
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
    We render normal maps for `main_xml(1)` and `mainDiffMat_xml(1)`, and depth maps for `main_xml(1)`.

4. **[Mask](https://drive.google.com/drive/folders/1GkLnZTIOSeIeewol0wyllP3opdMxdH1i?usp=sharing)** and **[Mask.zip](https://drive.google.com/file/d/1TxejwEi7v7wxTSJjXZnTAX2WhMK1MZF2/view?usp=sharing)**: The 480 × 460 grey scale mask `immask_*.png` for light sources. The pixel value 0 represents the region of environment maps. The pixel value 0.5 represents the region of lamps. Otherwise, the pixel value will be 1. We render the ground-truth masks for `main_xml(1)` and `mainDiffLight_xml(1)`.

5. **[SVLighting](https://drive.google.com/drive/folders/12Y13IVJcxeu8IblB9OIfQDiy3CEIQHeI?usp=sharing)** and **[SVLighting.zip](https://drive.google.com/file/d/1ibDk2LOOPelZal13E0xdyLgzudNqB44D/view?usp=sharing)**: The (120 × 16) × (160 × 32) per-pixel environment maps `imenv_*.hdr`. The spatial resolution is 120 x 160 while the environment map resolution is 16 x 32. To read the per-pixel environment maps, we can use the following python commands.
    ```python
    # Read the envmap of resolution 1920 x 5120 x 3 in RGB format
    env = cv2.imread('imenv_1', -1)[:, :, ::-1]
    # Reshape and permute the per-pixel environment maps
    env = env.reshape(120, 16, 160, 32, 3)
    env = env.transpose(0, 2, 1, 3, 4)
    ```
    We render per-pixel environment maps for `main_xml(1)`, `mainDiffMat_xml(1)` and `mainDiffLight_xml(1)`. Since the total size of per-pixel environment maps is 4.0 TB, we do not provide an extra .zip format for downloading. Please consider using the tool [Rclone](https://rclone.org/) if you hope to download all the per-pixel environment maps.

6. **[SVSG](https://drive.google.com/drive/folders/10mTrrCQXmEC-4wvlQTh-hTBETu2x1SxN?usp=sharing)** and **[SVSG.zip](https://drive.google.com/file/d/1MltE_Hoyb1jNV4p3H6n7NogCWQ9cFFQ_/view?usp=sharing)**: The ground-truth spatially-varying spherical Gaussian (SG) parameters `imsgEnv_*.h5`, computed from this optimization [code](https://github.com/lzqsd/SphericalGaussianOptimization). We generate the ground-truth SG parameters for `main_xml(1)`, `mainDiffMat_xml(1)` and `mainDiffLight_xml(1)`. For the detailed format, please refer to the optimization [code](https://github.com/lzqsd/SphericalGaussianOptimization).

7. **[Shading](https://drive.google.com/drive/folders/1WJApmMLh0wM64fhkKA4L7iJot6Y86VXW?usp=sharing)** and **[Shading.zip](https://drive.google.com/file/d/1lZsGEyeeUbUl-i68nIMuuT85Sptc6XPi/view?usp=sharing)**: The 120 × 160 diffuse shading `imshading_*.hdr` computed by intergrating the per-pixel environment maps. We render shading for `main_xml(1)`, `mainDiffMat_xml(1)` and `mainDiffLight_xml(1)`.

8. **[SVLightingDirect](https://drive.google.com/drive/folders/16LgLOPg-E9cTMDnK-4ysS5FxA5aT4Jdz?usp=sharing)** and **[SVLightingDirect.zip](https://drive.google.com/file/d/13aBQ-1xDZ0kjt80KmfWvMmS76Q9Y0NSv/view?usp=sharing)**: The (30 × 16) × (40 × 32) per-pixel environment maps with direct illumination `imenvDirect_*.hdr` only. The spatial resolution is 30 × 40 while the environment maps resolution is 16 × 32. The direct per-pixel environment maps can be load the same way as the per-pixel environment maps. We only render direct per-pixel environment maps for `main_xml(1)` and `mainDiffLight_xml(1)` because the direct illumination of `mainDiffMat_xml(1)` is the same as `main_xml(1)`.

9. **[ShadingDirect](https://drive.google.com/drive/folders/1AiUeU0VsPvQlaBwEyqOkotDSTM1JsNZo?usp=sharing)** and **[ShadingDirect.zip](https://drive.google.com/file/d/1KMbo5e5lAPLztEiM301dPBPFQ97mWhTA/view?usp=sharing)**: The 120 × 160 direct shading `imshadingDirect_*.rgbe`. To load the direct shading, we can use the following python command.
    ```python
    im = cv2.imread('imshadingDirect_1.rgbe', -1)[:, :, ::-1]
    ```
    Again, we only render direct shading for `main_xml(1)` and `mainDiffLight_xml(1)`

10. **[SemanticLabel](https://drive.google.com/drive/folders/17bp8K8gCrcfoXi_RDrtIN_uCTq2MW6VG?usp=sharing)** and **[SemanticLabel.zip](https://drive.google.com/file/d/1GU5PgHrYg-N3kRla6dhBr-oT6LuOxSmp/view?usp=sharing)**: The 480 × 640 semantic segmentation label `imsemLabel_*.npy`. We provide semantic labels for 45 classes of commonly seen objects and layout for indoor scenes. The 45 classes can be found in `semanticLabels.txt`. We only render the semantic labels for `main_xml(1)`.

11. **[LightSource](https://drive.google.com/drive/folders/1mBX3yoe7XSuDCW7jLPfnT7xfJI6NRwLU?usp=sharing)** and **[LightSource.zip](https://drive.google.com/file/d/1L-oUvZ9bD89fcqUyl14qgIQaCZHE_obo/view?usp=sharing)**: The light source information, including geometry, shadow and direct shading of each light source. In each scene directory, `light_x` directory corresponds to `im_x.hdr`, where x = 0, 1, 2, 3 ... In each `light_x` directory, you will see files with numbers in their names. The numbers correspond to the light source ID, i.e. if the IDs are from 0 to 4, then there are 5 light sources in this scene.
    * **Geometry**: We provide geometry annotation for windows and lamps `box_*.dat` for `main_xml(1)` only. To read the annotation, we can use the following python commmands.
        ```python
        with open('box_0.dat', 'rb')  as fIn:
            info = pickle.load(fIn )
        ```
        There are 3 items saved in the dictionary, which we list blow.
        * **isWindow**: True if the light source is a window, false if the light source is a lamp.
        * **box3D**: The 3D bounding box of the light source, including center `center`, orientation `xAxis, yAxis, zAxis` and size `xLen, yLen, zLen`.
        * **box2D**: The 2D bounding box of the light source on the image plane `x1, y1, x2, y2`.
    * **Mask**: The 120 × 160 2D binary masks for light sources `mask*.png`. We only provide the masks for `main_xml(1)`.
    * **Direct shading**: The 120 × 160 direct shading for each light source `imDS*.rgbe`. We provide the direction shading for `main_xml(1)` and `mainDiffLight_xml(1)`.
    * **Direct shading without occlusion**: The 120 × 160 direct shading without occlusion for each light source `imNoOcclu*.rgbe`. We provide the direction shading for `main_xml(1)` and `mainDiffLight_xml(1)`.
    * **Shadow**: The 120 × 160 shadow maps for each light source `imShadow*.png`. We render the shadow map for `main_xml(1)` only.

