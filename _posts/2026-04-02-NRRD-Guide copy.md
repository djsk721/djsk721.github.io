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
