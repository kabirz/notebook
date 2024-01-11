### 安装包
```shell
pip install pycocotools
```

数据集下载地址：[COCO - Common Objects in Context](https://cocodataset.org/#download)
cocoapi中
#### annotation
指的是对图像中目标物体的标注信息，通常包括目标的类别、边界框位置、遮挡情况等。这些标注信息对于训练和评估目标检测、图像分割等模型非常重要。
#### category
指的是物体的分类
#### image
指的图像
##### getAnnIds
函数是通过指定category来获取对于的annotation的id，如
```python
from pycocotools.coco import COCO
coco = COCO('./instances_val2017.json')
person_category_id = coco.getCatIds(catNms=['person']) # 获取人类的类别的id，这里是1
person_annotations_ids = coco.getAnnIds(catIds=person_category_id) # 通过指定类别id获取对应的标注id
person_annotations_ids = coco.getAnnIds(252844) # 通过image id来指定，不给参数代表所有标注信息
person_annotations = coco.loadAnns(person_annotations_ids) # 获取"person"类别的标注信息
imgIds = coco.getImgIds(catIds=person_category_id) # 根据指定的类别id获取对应image的id


person_info = coco.loadCats(person_category_id) # 根据人类类别id获取人类相关信息
# output： [{'supercategory': 'person', 'id': 1, 'name': 'person'}]
img = coco.loadImgs([250282, 255664]) # 这里的参数可以是一个imgIds, 也可以是多个imgIds组成的列表
# output: 输出是这个图相关的信息(大小和链接等等)
使用skimage.io 可以在线导入图
io.imread(img[0]['coco_url'])
```
这里的返回值`person_annotations_ids`是一系列人类对于的标注信息的id
`person_annotations`是一个含有人类信息的标注信息