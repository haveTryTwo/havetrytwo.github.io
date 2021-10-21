## 背景
我们在实际工作中，经常会遇到pb转json的场景，这里总结下当前pbtojson的思路和具体实现，可以为实际工作提供需要；

## 思路
### 1. PBToJson
采用PB的反射机制对PB协议进行遍历，获取每个字段数据，然后判断字段类型在进行数据处理；
- 注意：
   - 对PB的extension字段没有处理，**因为extension字段可以和非extension中字段重名等，那么数据就无法区分到底extension还是非extension**，实际使用中这里要注意；（关于extension字段获取参考 protobu => "FAQ(2) - pbtojson")
   - 需要判断字段是否repeated: 如果是的话，转json要采用数组类型处理
```
1. 获取pb的反射信息(Descriptor, Reflection)
2. 遍历字段，判断类型，然后调用json进行设置
```
   
### 2. JsonToPB
首先要遍历Json，转为内存结构体（当前采用rapidjson），然后遍历内存结构体，再通过反射从PB中获取对应字段信息，通过判断字段类型再从Json的内存结构体获取对应类型的数据；
- 说明：之所以在遍历内存结构体时要从PB中获取字段类型，因为PB是schema已经确定的，即如果json类型不满足该schema类型则不允许处理；
- 注意：
    - 判断字段是否repeated: 如果是的话，转PB需要采用repeated的方式处理；
```
1. 加载json到内存
2. 遍历内存对象，并通过PB找到对应数据，通过PB获取类型，再从json获取值，并进行设置
```
    
    
## 说明
1. 字段名不区分大小写：protobuf在将proto转换为c++文件时，会将大小写统一处理为小写来定义名称，所以在JsonToPB的时候要忽略字段名大小写；

## 压测
机型：MacBook Pro (15-inch, 2018)， 2.9GHz, intel core i9
压测方式：单核CPU 100%

|  | 长度 | 次数 | 耗时 | QPS |
| --- | --- | --- | --- | --- |
| PBToJson | 700B | 1W | 0.25s | 4W/s |
| JsonToPB | 700B | 1W | 0.33s | 3W/s |
    

## 参考
1. 代码参考： [pb_to_json.cc](https://github.com/haveTryTwo/csutil/blob/master/proto/pb_to_json.cc)
