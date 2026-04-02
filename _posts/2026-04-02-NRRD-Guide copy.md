---
title: "PyNRRD: NRRD 의료영상 데이터 파이썬 처리 가이드"
date: 2026-04-02
description: Python 패키지인 PyNRRD를 활용하여 NRRD 파일을 읽고 메타데이터를 확인하며, 데이터 변환 및 저장까지 가능한 실전 예제와 팁 안내
categories: [python]
tags: [nrrd, pynrrd, 의료영상, medical imaging, 파이썬, 이미지분석]
---

<aside>

 **NRRD(Nearly Raw Raster Data)** format: 데이터를 거의 원시(raw) 형태로 저장하면서 필요한 메타데이터와 함께 제공

</aside>

- 공식 문서: https://pypi.org/project/pynrrd/

```python
import nrrd
import numpy as np

# NRRD 파일 읽기
file_path = 'path/to/your/file.nrrd'
data, header = nrrd.read(file_path)

# NumPy 배열로 변환된 데이터 확인
print("Data shape:", data.shape)
print("Header information:", header)

# NRRD 파일 쓰기
new_data = np.random.random((10, 10, 10))
new_file_path = 'path'
nrrd.write(new_file_path, new_data)
```
