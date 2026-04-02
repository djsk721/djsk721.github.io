<aside>

💡 **Dicom** format: 의료영상정보를 처리하기 위한 국제 표준 이미지 형식으로, 헤더 메타 데이터(이미지 설명 데이터, 환자 데이터)+이미지로 구성됨

</aside>

---

## **PyDicom**

- 공식문서 페이지 [https://pydicom.github.io/7](https://pydicom.github.io/7)

```python
pip install pydicom

import pydicom
```

---

### 데이터 읽기 (.dcmread)

```python
# key-value 쌍을 가지는 dict 형태로 출력됨 
dcm_data = pydicom.dcmread('path')
dcm_data
```

- key
- value

```python
# 키워드를 통해 표준 요소에 액세스할 수 있음 

dcm_data['SOPInstanceUID']
dcm_data.SOPInstanceUID

# 요소는 multi-valued일 수 있음(VM > 1)
dcm_data.ImageType

dcm_data.file_meta
```

![Dicom header 예시](/assets/images/posts/2026-04-02-Pydicom-Guide/dicom.png)

---

### load option

Img array가 무거우므로 로드 시 out of memory 발생할 수 있음 → header만 불러와 탐색하기

```python
# option 
stop_before_pixels=True
specific_tags=['header1', 'header2', ...]
```

---

### img load

```python
import os
import pydicom
import matplotlib.pyplot as plt

def display_dicom_images(dcm_folder):
    # 리스트에 dcm 파일들을 저장
    dcm_files = [f for f in os.listdir(dcm_folder) if f.endswith(".dcm")]

    # 모든 DICOM 파일에 대해 루프
    for dcm_file in dcm_files:
        # DICOM 파일 경로 생성
        dcm_path = os.path.join(dcm_folder, dcm_file)

        # Pydicom을 사용하여 DICOM 파일 읽기
        dicom = pydicom.dcmread(dcm_path)

        # DICOM 이미지 추출
        image_array = dicom.pixel_array

        # 이미지 출력
        plt.imshow(image_array, cmap='gray')
        plt.title(f"DICOM Image: {dcm_file}")
        plt.show()

folder_path = 'path'
display_dicom_images(folder_path)
```
