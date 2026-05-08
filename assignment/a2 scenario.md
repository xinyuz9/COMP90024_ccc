# Scenario 1:
Cost-of-living pressure
- 高油价如何进入澳洲人的日常生活成本讨论？
- 是否伴随更多负面情绪和生活压力表达？

# Scenario 2:
Blame attribution
- 公众将高油价归因于谁？
- 全球冲突、政府、fuel excise、零售商、企业贪婪之间的归因比例是否变化？

# Scenario 3:
Transport adaptation
- 面对高油价，公众是否开始讨论替代出行？
- EV / BYD / public transport / driving less 的讨论是环保驱动，还是经济压力驱动？

# Pipeline
```
当前不使用 external API 的 pipeline：

1. 使用 Fission functions 抓取数据：
   - BlueSky posts
   - Reddit posts
   - GDELT news
   - 官方油价数据

2. 将原始数据存入 Elasticsearch：
   - raw_posts
   - raw_news
   - raw_fuel_prices

3. 对 social posts 做本地轻量 NLP 分析：
   - 用 keyword / domain rules 判断 relevance、main concern、attribution target、transport adaptation
   - 用 VADER 做 sentiment analysis

4. 将分析结果写回 Elasticsearch：
   - post_analysis
   - daily_social_aggregates

5. 对官方油价数据做处理：
   - 统一 NSW / VIC / QLD / WA 的字段
   - 统一 fuel type，例如 U91 / ULP / regular unleaded
   - 计算每日各州平均油价
   - 计算人口加权的 major-state fuel price indicator

6. 将油价聚合结果写入 Elasticsearch：
   - state_daily_fuel_prices
   - population_weighted_fuel_index

7. 后端通过 REST API 提供数据：
   - social sentiment trend
   - concern category trend
   - attribution target trend
   - adaptation trend
   - daily fuel price trend
   - fuel price indicator vs social discussion

8. Jupyter Notebook 作为前端：
   - 可视化官方油价趋势
   - 可视化帖子数量和情绪变化
   - 展示公众主要关切、责任归因和出行替代讨论
```


# 官方数据:

**NSW、VIC、QLD、WA 四个主要州的官方燃油价格数据** 构造一个高频的 **major-state fuel price indicator**。先从各州官方数据源获取 station-level fuel prices，统一字段和 fuel type，例如 U91 / regular unleaded，然后按 `state + date + fuel_type` 聚合出每个州的每日平均油价。之后再根据四州人口权重计算一个人口加权综合油价指标。这个指标不是官方全国平均油价，而是我们自己构造的 **population-weighted major-state retail fuel price proxy**，用于给社交媒体分析提供每日官方价格背景。

| State | Dataset                            | 格式 / 入口                             | 主要用途                                                                                                                                                                                                                                                                                                                                                                                                                            |
| ----- | ---------------------------------- | ----------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| NSW   | **NSW FuelCheck Price History**    | CSV / Data.NSW                      | NSW station-level fuel price history。March 2026 CSV 包含 `ServiceStationName`, `Address`, `Suburb`, `Postcode`, `Brand`, `FuelCode`, `PriceUpdatedDate`, `Price` 等字段。([Data.NSW](https://data.nsw.gov.au/data/en/dataset/a97a46fc-2bdd-4b90-ac7f-0cb1e8d7ac3b/resource/5576690a-2d64-4c61-a6a8-fa2975fc9d4e?view_id=9c16868e-3732-4ccb-aa78-ca88d17dd1ac&utm_source=chatgpt.com "FuelCheck Price History March 2026 - Data.NSW")) |
| VIC   | **Servo Saver Public API**         | JSON API / Service Victoria         | Victoria registered service stations 的 fuel price data；官方说明它是 Service Victoria 提供的 public API。([VicServices](https://service.vic.gov.au/find-services/transport-and-driving/servo-saver/help-centre/servo-saver-public-api?utm_source=chatgpt.com "Servo Saver Public API"))                                                                                                                                                    |
| QLD   | **Fuel Price Reporting 2026**      | CSV + API / data.qld.gov.au         | Queensland service stations 的 fuel prices，官方页面说明 available as CSV and API。([Queensland Government Data](https://www.data.qld.gov.au/dataset/fuel-price-reporting-2026?utm_source=chatgpt.com "Fuel price reporting 2026 - Dataset"))                                                                                                                                                                                            |
| WA    | **FuelWatch Historic Fuel Prices** | CSV / historic download / FuelWatch | WA FuelWatch scheme 下 fuel retailers 提供的 daily fuel prices，自 2001 年以来覆盖；适合做每日官方零售油价。([Data WA](https://catalogue.data.wa.gov.au/dataset/fuelwatch-historic-fuel-prices/resource/903a3bfb-b8ea-4cff-8c24-47a05d40b112?utm_source=chatgpt.com "FuelWatch Historic Fuel Prices"))                                                                                                                                                  |



## Data Analysis

- **Step 1: 泛向摄入 (Broad Ingestion / Raw Data)**
    
    - 抛弃精准长尾词搜索。直接从 GDELT、Bluesky 和 Mastodon 抓取带有 `Australia` 标签或本地节点的**全量/宽泛数据**。
        
    - _目的：_ 撑起系统吞吐量（达到 Tens of GBs），证明云架构的数据承载能力。
        
- **Step 2: 降维清洗 (Schema Cleaning)**
    
    - 展平原始 JSON，**丢弃覆盖率低于 1% 的垃圾字段**，但强制保留核心语义字段（如 `author.did`, `timestamp`）。
        
    - _目的：_ 防止 ES Mapping 爆炸，大幅压缩数据体积。
        
- **Step 3: NLP 过滤与情感分析 (Target Analysis)**
    
    - 注入 48 个与“生活成本/高油价/出行”相关的核心关键词矩阵进行严格过滤。
        
    - 对命中关键词的文本使用 **VADER 提取情感分数 (Sentiment Score)**。

