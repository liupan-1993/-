```python
import numpy as np
import pandas as pd
```


```python
admissions = pd.read_csv('binary.csv')
```


```python
admissions.head()
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>admit</th>
      <th>gre</th>
      <th>gpa</th>
      <th>rank</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>0</td>
      <td>380</td>
      <td>3.61</td>
      <td>3</td>
    </tr>
    <tr>
      <td>1</td>
      <td>1</td>
      <td>660</td>
      <td>3.67</td>
      <td>3</td>
    </tr>
    <tr>
      <td>2</td>
      <td>1</td>
      <td>800</td>
      <td>4.00</td>
      <td>1</td>
    </tr>
    <tr>
      <td>3</td>
      <td>1</td>
      <td>640</td>
      <td>3.19</td>
      <td>4</td>
    </tr>
    <tr>
      <td>4</td>
      <td>0</td>
      <td>520</td>
      <td>2.93</td>
      <td>4</td>
    </tr>
  </tbody>
</table>
</div>




```python
#特征提取
data = pd.concat([admissions, pd.get_dummies(admissions['rank'],prefix = 'rank')],axis=1).drop('rank',axis = 1)
```


```python
#标准化方法
def normalized(data):
    return (data-data.mean())/data.std()
```


```python
#标准化‘gre’和‘gpa’
for field in ['gre', 'gpa']:
    mean, std = data[field].mean(), data[field].std()
    data.loc[:,field] = (data[field]-mean)/std
```


```python
#随机的选择训练集，和验证集
np.random.seed(21)
sample = np.random.choice(data.index, size = int(len(data)*0.9), replace = False)
data, test_data = data.loc[sample],data.drop(sample)
```


```python
#分离数据为特征和结果
features, targets = data.drop('admit',axis = 1), data['admit']
test_feature, test_targets = test_data.drop('admit',axis = 1),test_data['admit']
```


```python
#sigmoid函数
def sigmoid(x):
    return 1/(1+np.exp(-x))
```


```python
#隐藏层特征数，训练次数，学习率
n_hidden = 2
epochs = 900
learn_rate = 0.005
```


```python
n_records,n_features = features.shape
last_loss = None
```


```python
#初始化权重
np.random.seed(21)
weight_input_hidden = np.random.normal(scale=1 / n_features ** .5,size = (n_features,n_hidden))
weight_hidden_output = np.random.normal(scale=1 / n_features ** .5,size = n_hidden)
```


```python
for e in range(epochs):
    #初始化权重变化值
    del_w_input_hidden = np.zeros(weight_input_hidden.shape)
    del_w_hidden_output = np.zeros(weight_hidden_output.shape)
    for x,y in zip(features.values,targets):
  
        
        #正向传播输入-隐藏层
        hidden_input = np.dot(x,weight_input_hidden)
        
        #激活函数
        hidden_output = sigmoid(hidden_input)
        
        #正向传播隐藏层-输出
        output = sigmoid(np.dot(hidden_output,weight_hidden_output))
        
        #误差
        error = y-output
        
        #反向传播输出-隐藏层
        output_error_term = error*output*(1-output)
        
        #反向传播隐藏层-输入
        hidden_error = np.dot(output_error_term,weight_hidden_output)
        hidden_error_term = hidden_error*hidden_output*(1-hidden_output)
        
        #隐藏层-输出权重变化
        del_w_hidden_output += learn_rate*hidden_output*output_error_term/n_records
        #输入-隐藏层权重变化
        del_w_input_hidden += learn_rate*x[:,None]*hidden_error_term/n_records
        #更新权重变化值
    weight_input_hidden += del_w_input_hidden
    weight_hidden_output += del_w_hidden_output

```


```python
hidden = sigmoid(np.dot(test_feature, weight_input_hidden))
out = sigmoid(np.dot(hidden, weight_hidden_output))
predictions = out > 0.5
accuracy = np.mean(predictions == test_targets)
print("Prediction accuracy: {:.3f}".format(accuracy))
```

    Prediction accuracy: 0.725
    
