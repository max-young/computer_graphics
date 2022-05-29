### 23.7 Depth Culling, Testing, and Buffering

假设我们要渲染一张纸铺在桌子上的场景, 纸只比桌子高一点点, 由于精度的限制, 可能会造成桌子戳破纸的情况, 这种现象称为z-fighting.  

如果视角是在桌子的正上方, 提高z-buffer的精度就可以避免z-fighting了.  
如果是从侧方观察, 由于视角的关系, 需要做一些数学转换, 提高远处的精度, 来避免z-fighting.