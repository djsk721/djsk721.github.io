<aside>
💡 **NiftTI** format: 헤더+3D array로 구성되며 dicom 이미지 프레임을 합친 것

</aside>

### Nibabel

공식문서 페이지 https://nipy.org/nibabel/

- 설치

```python
pip install nibabel
import nibabel as nib
```

- 데이터 읽기 (nib.load)
    
    ```python
     
    ```
    

- Affine and Header
    - affine: 4*4 array 형태로 voxel space에서 reference space으로의 변환을 설명함
    - header: 이미지의 모든 메타데이터를 저장하는 nibabel 구조로, 직접 쿼리할 수 있음

```python
nib.aff2axcodes(affine)
img.header['descrip']
```

새 이미지를 만들었을 때 원본과 파생물이 일치하는지 확인하기 위해 Affine과 Header를 탐색할 수 있음 

- img load

```python
def display_slices(nii_file_path, num_slices=5):
    # NIfTI 파일 읽기
    nii_img = nib.load(nii_file_path)

    # NIfTI 데이터 얻기
    nii_data = nii_img.get_fdata()

    # 데이터 크기 확인
    num_slices_available = nii_data.shape[-1]

    # 출력할 슬라이스 개수 선택
    num_slices_to_display = min(num_slices, num_slices_available)

    # 선택된 슬라이스를 플로팅
    fig, axes = plt.subplots(1, num_slices_to_display, figsize=(15, 5))

    for i in range(num_slices_to_display):
        # 슬라이스 번호 계산
        slice_index = i * (num_slices_available // num_slices_to_display)
        
        # 슬라이스 플로팅
        axes[i].imshow(nii_data[:, :, slice_index], cmap='gray')
        axes[i].set_title(f"Slice {slice_index}")
        axes[i].axis('off')

    plt.show()

nii_file_path = 'path'
display_slices(nii_file_path)
```
