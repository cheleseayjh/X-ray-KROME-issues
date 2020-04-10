## KROME running logic

### Linux 命令行运行

- KROME的逻辑（详见KROME-Wiki）

  ​	向主程序输入反应网，将反应逐项输入build的subs.f90等文件，并配套写入有关的function（如heating等），具体的过程不完全清楚，但重要信息一般在sub.f90, ode.f90, getphys.f90中。

  ​	写入./test命令，将build中的test.f90录入主程序，逐步地调取所需函数计算反应网的演化。

  ​	最终得到结果，可借助plot进行画图。

- 存在的问题

  ​	由于程序本身的原因，在./test过程中会对大量文件进行覆写，其中包含我们的核心目标X-ray ionization的有关内容（H、He柱密度表），导致我们无法进行有效的改动。



### 外部干预（现用Python实现）

- 目的

  ​	实现我们所给条件下各种所需信息的“冻结”，即不能使他们被KROME自身的覆写所覆盖。

- 实现

  ​	当修改反应网后，用KROME进行初始化直到./test前（即到 make gfortran）。可以单独进行一个project便于操控：

  ```fortran
  ./krome -project=your_project -n=networks/your_network -noRecCheck
  ```

  ​	随后将新的build文件夹中所有文件上传至AGN_pc，替换先前文件，得到“冻结”后的所需信息。AGN.ipynb中通过覆写test.f90来实现不同的环境（即初始条件等，但都不会改变反应网，从而不改变计算程序）

- details

  - subs.f90中要加入除了H、He的X-ray ionization，举例如下：

    ```fortran
        !CO -> C + O
        k(31) = k(31) + 4.7093110781728 * k(4430)*0.8
    
        !CO -> CO+ + E
        k(32) = k(32) + 4.7093110781728 * k(4430)*0.2
    ```

  - ode.f90中 dn(idx_H) 有两个，要把它们接在一起
  
  - 将KROME生成的species.gps 放入/data中，并print python文件以备查找和打印图表
  
    ​	<u>不要忘记设置 nkrome = 0</u>
  
  - AGN.ipynb中有关于柱密度转换的改动：
  
    ```python
    def AGN(...):
    	...
        with open('krome_getphys.f90', 'r') as f:
        lines = f.readlines()
        lines[3715] = 'num2col = max(ncalc,1d-40)*{}\n'.format(str(NH / 2e4))
        ...
    ```
  
    ​	这里的行数在反应网更改后就需要更改，一定注意！（通过观察getphys.f90来监控）
    
  - 在Wakelam，2008的反应网（react_cloud）中，gas phase 的物质记为
  
    ```fortran
    CH4O,CH4O+;C2H2N,C2H2N+;C2H3N,C2H3N+
    ```
  
    但在我们的grain process中，与dust相对应的gas中物质（即比如从dust上desorption的物质）被记为
  
    ```fortran
    CH3OH;CH2CN;CH3CN
    ```
  
    并且其相应的正离子被记为
    
    ```fortran
    CH4O+,CH2CN+,CH3CN+ (our case); CH4O+,C2H2N+,C2H3N+ (Wakelam, 2008)
    ```
    
    总结来看，在Wakelam+grain process 与Chang's network 中分别有这些物质：
    
    ```fortran
    Wakelam: CH3OH,CH4O,CH4O+; CH2CN,C2H2N,C2H2N+; CH3CN,C2H3N,C2H3N+
    Chang: CH3OH,CH4O+; CH2CN,CH2CN+; CH3CN,CH3CN+
    ```
    
     