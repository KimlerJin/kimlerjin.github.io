---
layout: post
title: "最近用cursor写代码，写了个根据高德地图爬附近企业信息的脚本"
date: 2025-02-08
categories: blog
---

# 高德地图 API 周边企业查询脚本

以下是一个 Python 脚本，用于从 Excel 文件中读取产业园地址，并通过高德地图 API 查询周边企业信息。

## 代码实现

```python
import pandas as pd
import requests
import logging

# 配置日志输出
logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s: %(message)s')
logger = logging.getLogger()

# 高德地图 API Key
api_key = '0b56e096bf6ccd030318556934966ffa'  # 请确保这是有效的key(不要用我这个，已经用完了)

def main():
    try:
        # 读取 Excel 文件
        excel_file = '工业园名单.xlsx'
        logger.info(f"开始读取Excel文件: {excel_file}")
        df = pd.read_excel(excel_file, engine='openpyxl')  # 显式指定引擎
        
        # 检查是否存在地址列
        if '地址' not in df.columns:
            raise ValueError("Excel文件中缺少'地址'列，请检查列名是否一致")
        
        logger.info(f"成功读取 {len(df)} 条地址数据")
        results = []

        # 遍历每个地址
        for index, row in df.iterrows():
            address = row['地址']
            if pd.isna(address) or not address.strip():
                logger.warning(f"第 {index+1} 行地址为空，已跳过")
                continue
            
            logger.info(f"正在处理第 {index+1}/{len(df)} 条地址: {address}")
            
            # ---------------------- 地理编码API ----------------------
            try:
                geocode_url = f'https://restapi.amap.com/v3/geocode/geo?address={address}&key={api_key}'
                logger.debug(f"地理编码请求URL: {geocode_url}")
                geocode_response = requests.get(geocode_url, timeout=10)
                geocode_response.raise_for_status()  # 检查HTTP状态码
                geo_data = geocode_response.json()
                
                if geo_data['status'] != '1':
                    logger.error(f"地理编码API错误: {geo_data.get('info', '未知错误')}")
                    continue
                    
                if not geo_data['geocodes']:
                    logger.warning(f"地址解析失败: {address}")
                    continue
                    
                location = geo_data['geocodes'][0]['location']
                logger.info(f"成功获取坐标: {location}")
                
            except Exception as e:
                logger.error(f"地理编码处理异常: {str(e)}")
                continue
            
            # ---------------------- 周边搜索API ----------------------
            try:
                search_url = f'https://restapi.amap.com/v3/place/around?key={api_key}&location={location}&radius=1000&types=170000'
                logger.debug(f"周边搜索请求URL: {search_url}")
                search_response = requests.get(search_url, timeout=10)
                search_response.raise_for_status()
                search_data = search_response.json()
                
                if search_data['status'] != '1':
                    logger.error(f"周边搜索API错误: {search_data.get('info', '未知错误')}")
                    continue
                
                if int(search_data.get('count', 0)) == 0:
                    logger.warning("未找到周边企业信息")
                    continue
                    
                for poi in search_data['pois']:
                    results.append({
                        '产业园地址': address,
                        '企业名称': poi.get('name', ''),
                        '企业地址': poi.get('address', ''),
                        '联系方式': poi.get('tel', '无')
                    })
                logger.info(f"本地址找到 {len(search_data['pois'])} 家企业")
                
            except Exception as e:
                logger.error(f"周边搜索处理异常: {str(e)}")
                continue

        # 保存结果
        if len(results) == 0:
            logger.warning("没有找到任何企业信息，请检查输入地址或API响应")
            return
            
        result_df = pd.DataFrame(results)
        output_file = '周边企业信息.xlsx'
        result_df.to_excel(output_file, index=False)
        logger.info(f"成功保存 {len(result_df)} 条数据到 {output_file}")
        
    except Exception as e:
        logger.error(f"主程序异常: {str(e)}")
        raise

if __name__ == '__main__':
    main()
```
# 使用说明


## 1.确保已安装以下 Python 库：


```python

pip install pandas requests openpyxl
```
## 2.将高德地图 API Key 替换为你的有效 Key。(需要先去申请高德地图开发者账号)
``` 
https://console.amap.com/dev/index
``` 
并且key类型勾选为web服务

## 3.将 Excel 文件路径修改为你的文件路径。
## 4.运行脚本 
```
python script_name.py
```