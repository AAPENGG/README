# STHTreeMapReduce 分布式处理程序

## 概述
`STHTreeMapReduce` 是一个基于 Apache Hadoop 的 MapReduce 程序，用于处理地理空间数据。该程序从输入文件中读取数据，根据数据的经纬度信息判断其是否位于指定的矩形区域内，并对数据进行相应的处理和分析。程序的主要功能包括：
- 过滤无效数据（如空行、格式不正确的数据）。
- 确定数据点所在的矩形区域。
- 计算每个矩形区域内数据的时间范围和哈希码信息。
- 计算哈希码信息对应的树的最大层级。

## 输入数据格式
输入数据文件应为文本文件，每行数据包含至少 4 个字段，字段之间用逗号分隔，格式如下：
timestamp,latitude,longitude,hashCode
- `timestamp`：时间戳，格式为 ISO 8601 日期时间格式（例如：`2023-01-01T12:00:00Z`）。
- `latitude`：纬度，为一个浮点数。
- `longitude`：经度，为一个浮点数。
- `hashCode`：哈希码，为一个字符串。

## 输出数据格式
输出数据文件包含每个矩形区域的信息，包括：
- 数据点的详细信息（经纬度、时间戳、哈希码）。
- 矩形区域的时间范围（左时间和右时间）。
- 所有哈希码信息及对应的树的最大层级。

## 代码结构
### Mapper 类：`STHTreeMapper`
`STHTreeMapper` 类继承自 `Mapper<Object, Text, Text, Text>`，负责读取输入数据，过滤无效数据，并将数据映射到相应的矩形区域。主要步骤如下：
1. 读取输入数据行，跳过空行和格式不正确的数据。
2. 解析数据行中的时间戳、经纬度和哈希码。
3. 判断数据点是否位于指定的矩形区域内，生成对应的输出键（矩形区域信息和时间戳）。
4. 将输出键和数据点信息（经纬度和哈希码）写入上下文。

### Reducer 类：`STHTreeReducer`
`STHTreeReducer` 类继承自 `Reducer<Text, Text, Text, NullWritable>`，负责对 Mapper 输出的数据进行归约处理。主要步骤如下：
1. 遍历每个键（矩形区域信息和时间戳）对应的所有值，过滤格式错误的值。
2. 解析数据点信息，提取经纬度和哈希码。
3. 计算矩形区域内数据的时间范围（左时间和右时间）。
4. 收集所有哈希码信息，计算哈希码信息对应的树的最大层级。
5. 将矩形区域信息、时间范围、哈希码信息和树的最大层级写入上下文。

### 主函数：`main`
`main` 函数负责配置和运行 MapReduce 作业，主要步骤如下：
1. 检查命令行参数，确保输入和输出路径已提供。
2. 配置作业的基本信息，如作业名称、Mapper 类、Reducer 类等。
3. 设置作业的输入和输出路径。
4. 提交作业并等待作业完成。

## 配置和运行
### 环境要求
- Apache Hadoop 集群环境。
- Java 开发环境（JDK 8 或更高版本）。

### 编译和打包
将 `STHTreeMapReduce.java` 编译并打包成 JAR 文件：
```sh
javac -classpath $(hadoop classpath) STHTreeMapReduce.java
jar cf sthtreemapreduce.jar STHTreeMapReduce*.class
hadoop jar sthtreemapreduce.jar com.STHTreeMapReduce <input_path> <output_path>
