
  本文介绍基于**Python**语言，逐一读取大量`.nc`格式的多时相栅格文件，导出其中所具有的全部时间信息的方法。


  `.nc`是`NetCDF`（Network Common Data Form）文件的扩展名，表示一种常用的科学数据存储格式。`NetCDF`是一种自描述的、可移植的二进制文件格式，用于存储科学和工程领域的大型数据集；由于其自身的特性，`.nc`数据被广泛应用于气象学、海洋学、地球科学、气候研究、大气科学、地理信息系统等领域。


  首先，明确一下本文的需求。现在有一个文件夹，其中具有大量的`.nc`格式的栅格文件，如下图所示。


![image](https://img2024.cnblogs.com/blog/3080295/202410/3080295-20241022170108717-2075121586.png)


  其中，每一个`.nc`格式的文件都具有**多个时相**（或者说是**多个维度**），而**不仅仅**只是一个时相。我们希望，读取这个文件夹中的全部`.nc`格式文件，并获取其所表示的每一个时相。


  明确了需求后，我们就可以开始具体的操作。首先，本文所需用到的代码如下。



```
# -*- coding: utf-8 -*-
"""
Created on Sun Dec 31 20:28:03 2023

@author: fkxxgis
"""

import os
import netCDF4
from netCDF4 import Dataset

def list_nc_dates(folder_path):
    nc_dates = []

    for file_name in os.listdir(folder_path):
        if file_name.endswith(".nc"):
            file_path = os.path.join(folder_path, file_name)
            try:
                dataset = Dataset(file_path)
                time_var = dataset.variables["time"]
                time_values = time_var[:]
                time_units = time_var.units
                time_calendar = time_var.calendar

                dates = []
                for value in time_values:
                    date = netCDF4.num2date(value, units=time_units, calendar=time_calendar)
                    dates.append(date.strftime("%Y-%m-%d %H:%M:%S"))

                nc_dates.append((file_name, dates))
            except Exception as e:
                print(f"Error reading file {file_name}: {str(e)}")

    return nc_dates

folder_path = "F:/Data_Reflectance_Rec/soil_1"
nc_dates = list_nc_dates(folder_path)

for nc_file, dates in nc_dates:
    for date in dates:
        print(date)

```

  这段代码整体思路也很明确。


  首先，我们导入所需的模块。在这里，需要导入**Python**的`os`模块，用于处理文件和文件夹路径操作；同时导入`netCDF4`库，并接着从`netCDF4`库中导入`Dataset`类，用于打开和读取`.nc`文件。在这里，如果需要配置`netCDF4`库，大家可以参考文章[配置h5py、netCDF4库的方法：Anaconda环境](https://github.com)。


  接下来，我们定义了一个名为`list_nc_dates`的函数，接受一个文件夹路径作为参数。在函数中，首先创建一个空列表`nc_dates`，用于存储每个`.nc`文件及其对应的日期列表；随后，使用`os.listdir()`函数遍历文件夹中的所有文件，通过检查文件名是否以`.nc`结尾来筛选出`.nc`文件。紧接着，对于筛选出来的`.nc`文件，使用`os.path.join()`函数构建其完整路径。


  其次，使用`Dataset`类打开`.nc`文件，并将打开的文件对象赋值给`dataset`变量；随后，获取`.nc`文件的时间，在本文的`.nc`数据中，也就是名为`time`的变量，并将时间变量的值读取到`time_values`变量中。接下来，分别获取时间变量的单位与时间类型。


  随后，我们创建一个空列表`dates`，用于存储日期字符串。遍历时间变量的每个值，使用`netCDF4.num2date()`函数将时间值转换为日期对象。紧接着，将日期对象转换为指定格式的字符串，并将其添加到`dates`列表中。此外，这里还将`.nc`文件名和对应的日期列表作为元组添加到`nc_dates`列表中，方便我们后期对日期的核对。函数的最后，返回包含每个`.nc`文件及其对应日期的列表。


  在函数外部，我们设置文件夹路径，随后即可调用`list_nc_dates`函数，将文件夹路径传递给它，并将返回的结果赋值给`nc_dates`变量。最后，通过循环，打印每个日期即可。


  执行上述代码，即可出现如下图所示的结果（结果很长，就截取一部分）。由于在本文中，每一个`.nc`格式文件的每一个维度（即每一个**时相**）都是精确到天的，所以下图天数后的时、分、秒都是`00`。当然，如果大家的`.nc`格式文件维度很多，时相打印出来的话也不好完全显示，所以可以考虑将时间信息导出为表格文件等；例如，可以将每一个`date`都放在**DataFrame**中，随后导出为`.csv`文件。


![](https://img2024.cnblogs.com/blog/3080295/202410/3080295-20241022170059730-706955529.png)


  至此，大功告成。


 本博客参考[FlowerCloud机场](https://hushicha.org)。转载请注明出处！
