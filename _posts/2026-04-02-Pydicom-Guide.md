---
title: "PyDicom: 의료영상(DICOM) 파이썬 처리 가이드"
date: 2026-04-02
description: Python 패키지인 PyDicom을 활용하여 DICOM 의료영상 파일을 읽고, 헤더 정보 추출 및 이미지를 시각화하는 실전 예제와 유용한 팁 안내
categories: [python]
tags: [dicom, pydicom, 의료영상, medical imaging, 파이썬, 이미지분석]
---


<aside>

**DICOM** 포맷: 의료영상 정보를 위해 사용되는 국제 표준 이미지 포맷.  
→ 헤더(meta 데이터: 이미지 설명, 환자 정보 등) + 바이너리 이미지 데이터로 구성됨

</aside>

---

## **PyDicom**

- 공식문서: [https://pydicom.github.io/](https://pydicom.github.io/)

```bash
pip install pydicom
```

```python
import pydicom
```

---

### DICOM 데이터 읽기 (.dcmread)

```python
# DICOM 파일을 읽어 들여 key-value 쌍(딕셔너리 형태) 반환
dcm_data = pydicom.dcmread('path/to/file.dcm')
print(dcm_data)
```

- **key**: 태그명
- **value**: 값

```python
# 키 또는 속성으로 주요 표준 요소 접근 가능
dcm_data['SOPInstanceUID']
dcm_data.SOPInstanceUID

# 단일/다중 값 요소(VM>1) 모두 지원
dcm_data.ImageType

# 파일 메타정보도 별도로 접근 가능
dcm_data.file_meta
```

![Dicom header 예시](/assets/images/posts/2026-04-02-Pydicom-Guide/dicom.png)

---

### 효율적인 로드 옵션

DICOM 파일은 이미지 array가 매우 클 수 있음  
→ OOM(메모리 부족) 방지를 위해 header 정보만 로드하거나 지정 태그만 일부 탐색 가능

```python
# 주요 옵션 예시
dcm_data = pydicom.dcmread(
    'path/to/file.dcm',
    stop_before_pixels=True,  # 이미지(pixels) 데이터 로드하지 않음
    specific_tags=['PatientID', 'StudyDate']  # 필요한 태그만 추출
)
```

---

### DICOM 이미지 배열 로드 및 시각화

```python
import os
import pydicom
import matplotlib.pyplot as plt

def display_dicom_images(dcm_folder):
    # DICOM(.dcm) 파일 목록 추출
    dcm_files = [f for f in os.listdir(dcm_folder) if f.endswith(".dcm")]

    # 모든 DICOM 파일 반복
    for dcm_file in dcm_files:
        dcm_path = os.path.join(dcm_folder, dcm_file)
        dicom = pydicom.dcmread(dcm_path)
        image_array = dicom.pixel_array  # 픽셀값 추출

        plt.imshow(image_array, cmap='gray')
        plt.title(f"DICOM Image: {dcm_file}")
        plt.axis('off')
        plt.show()

folder_path = 'path/to/dicom_folder'
display_dicom_images(folder_path)
```
