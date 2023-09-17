# Entity Matching For Tourism Dataset

## 1 Description

Imagine searching for hotels from two tourism websites. There should be many same hotels but with slightly different information in names or addresses. So, if we are conducting a big data processing task in the field of tourism, like building a knowledge graph, we are likely to face the problem that entities from different data sources are either same or different. When we need to integrate these data, it is important to find the same entities and remove duplicates. The figure below shows the main function of this entity matching model.

<!-- ![](Images/purpose.png) -->
 <div  align="center">  <img src="Images/purpose.png" width = "50%" height = "50%" /> </div>

The model is a Xgboost-based binary classification model. When encoded vectors of two entities come into the model, it will give the prediction label *same / different*.



## 2 Dataset and Features

Data in this project are from three sources - Ctrip, Tripadvisor, Mafengwo. And we take attractions in Hong Kong as a example. For each entity (attraction), we keep attributes of **name**, **address**, **location** (latitude and longitude), and **District**. The addresses will be segmented by a BERT-based model and divided into attributes **district**, **RoadInfo**, **RoomInfo**, and **Poi** (Details in address segment will not be displayed in this project). We have to mention that the attributes **District** and **district** are different. The former one comes from the tourism website and indicates the administrative district, whereas the latter one is a part of the address and includes administrative district / village / town / block. Moreover, the former one will only be used in entity pairing and the latter one will be one of the feature vectors.



## 3 Overall Process

The overall process will include 3 main stages - making training data / pairing application data, calculating feature vectors, model training / prediction, which is shown as below:

 <div  align="center">  <img src="Images/process.png" width = "50%" height = "50%" /> </div>

 Specifically, there is a strategy in entity paring to reduce the calculaation complexity. Clearly, entities in different districts must not be matched as a same one. In other words, we only need to pair entities from same districts. This strategy is shown as below:

  <div  align="center">  <img src="Images/pairing.png" width = "60%" height = "60%" /> </div>



## 4 Data Preprocessing

The preprocess of data is divided into 2 stages - making **training** data / pairing **application** data, as mentioned before (The vector calculation part is also included here). They correspond to the four folders 'train_xxx' and 'user_xxx' respectively. When both sets of data are processed, the model can be trained and predicted using **main.ipynb**.

### 4.1 training data

1. There are only two source data files in the initial data directory **'train_data'**
2. Utilising external address segment models, we get two segment results in address, namely **'xxx_addr_result.json'**
3. Run **'preprocess.ipynb'** in **'train_src'** to process the address segment results, simplify the information from source data, and combine them. So we get two processed data with segmented address, namely **'xxx_addr.json'**
4. Run **'matching.ipynb'** to pair the entities from 2 different sources by some strategies. Then we get positive and negative training data files.
    1. Calculate all embedding vectors for each strings in 2 source data in advance.
    2. Generate 200 pairs of entities with the most similar **Name** -> Manually label them (you need to find the corresponding .csv in the **temp** folder and add a column of 'label' to manually mark 1/0 (indicating match/mismatch)) -> Extract positive and negative data
    3. Same operation for **Distance**
    4. Same operation for **RoadInfo**
    5. Same operation for **RoomInfo**
5. 运行embedding.ipynb。对两个数据进行相似度计算，转换为DataFrame

## 其次是应用数据处理过程

1. 初始数据目录（user_data）下只有两个数据源文件
2. 通过外部的分词模型预测，得到两个源数据的地址分词结果，以“xxx_addr_result.json”命名
3. 运行代码目录（user_src）下的preprocess.ipynb。处理分词结果，精简源数据的信息，合并两者，得到两个加工后的带地址数据，以“xxx_addr.json”命名
4. 运行match&embedding.ipynb。将两个源的数据按照18个区进行同区两两匹配，然后对于每对数据进行词向量计算（使用应用数据预处理好的），得到“data.csv”文件

## 最终是模型训练与预测

1. 运行主目录下的main.ipynb。得到测试集预测结果“test_result.csv”和应用数据预测结果“result.json”
2. 运行visualization.ipynb。根据折线图决定最优阈值