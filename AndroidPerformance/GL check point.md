## 调查点：

1. 纹理是否压缩

   * 疑问：不同GPU支持的纹理格式不一样，需要对GPU做适配，工作量不小

   > 志文：壁纸或许可以做一些处理（go-launcher反编译代码未发现类似处理），理想情况下，可以减7-8M内存占用

2. Mipmap

   > 虽然Mipmap包会比原始图像多用掉33%的内存，但它不仅可以提升性能，也会提高图像质量
   >
   > > ARM Mali GPU Texture Compression Tool可以生成Mipmap

   * 结论：这个功能会更耗内存，适用于游戏场景，launcher不适用

3. 纹理参数设置

   > 基本越慢的函数，视觉质量越高
   >
   > 参数配置参考链接：https://www.khronos.org/registry/OpenGL-Refpages/es2.0/

4. RenderScript 3D 渲染


高斯模糊之类的优化