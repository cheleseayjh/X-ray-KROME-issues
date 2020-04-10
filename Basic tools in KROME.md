反应物、CR、温度等信息的初始化应位于build/test.f90之中

用户自定义的常数设置应该在test.f90中调取krome_user_common后再进行设置

（如果按照标准程序再构建反应网，由于test中没有输出，所以fort.66中的结果还是原先的结果——一定要在test中定义输出文件）：

```fortran
!output header
write(66,'(a)')
```

run  KROME 的语句：

```fortran
call krome(x(:),Tgas,dt) 
! x(:):abundance of species
! Tgas:gas temperature
! dt:time-step in intergral
```

J21xray 常数的设定：

```fortran
call krome_set_J21xray(const)
```

有关UV的flux，J21s 的设定格式：

```fortran
j21s = (/c1, c2, c3, c4/)
! ci is the seperate values of j21 (not continue variable)
```

