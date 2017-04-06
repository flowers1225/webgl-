# webgl-

《webgl编程指南》

## ch03

- 绘制流程

```
gl.createBuffer();
gl.bindBuffer();
gl.bufferData();
gl.getAttribLocation();
gl.vertexAttribPointer();
gl.enableVertexAttribArray();
gl.drawArrays();
```

- 旋转、位移
```
gl.getUniformLocation();
gl.uniform[1-4]f();
```

- 旋转、位移矩阵
```
//利用矩阵工具Matrix4()计算矩阵

new Matrix4();

.setRotate(angle, 0, 0, 1);
```

## ch04

```
//动画
requestAnimationFrame();
```

## ch05

- 点、颜色、大小绘制
```
let vertices = new Float32Array([
    -0.5, 0.5, 5,  1.0, 0.0, 0.0, 1.0,
    -0,   0,   10, 0.0, 1.0, 0.0, 1.0,
    0.5,  0.5, 20, 0.0, 0.0, 1.0, 1.0,
    0.3,  0,   10, 1.0, 0.0, 0.0, 1.0,
    0.5,  0.3, 10, 1.0, 0.0, 0.0, 1.0,
]);

//通过gl.vertexAttribPointer()的第5、6个参数来区分

let fSize = vertices.BYTES_PER_ELEMENT; //获取每个元素所占字节

//以大小为例：
// @fSize * 7 相邻顶点的间距
// @fSize * 2 缓冲区中偏移量
gl.vertexAttribPointer(a_size, 1, gl.FLOAT, false, fSize * 7, fSize * 2);

```

- 纹理

```
//顶点着色器

...
varying vec2 v_texture;
...

//片元着色器
precision mediump float; // 声明所有浮点数默认为中精度 必写 精度限定字

uniform sampler2D u_sampler; //纹理数据类型。这里是2d的所以定义sampler2D 取样器只能用uniform来定义
varying vec2 v_texture;

void main () {
    gl_FragColor = texture2D(u_sampler, v_texture);
}
```

>  在fragment里定义与vertex相同，填充操作由内部完成

1、顶点着色器接收顶点纹理坐标，光栅化后传入片元着色器

2、片元着色器根据片元的纹理坐标， 从图像中抽取出纹理颜色， 赋值当前片元

```
1、与绘制流程类似
2、
    gl.createTexture();
    gl.getUniformLocation();

    //图片加载成功后
    gl.pixelStorei();
    gl.activeTexture();
    gl.bindTexture();
    gl.texParameteri();
    gl.texImage2D();
    gl.uniform1i();
    gl.drawArrays();

```

## ch07

深度信息：在哪里、看哪里、视野宽、能看多远。

观察者 (观察方向、可是距离)

视点：观察者的位置。

视线：从视点出发沿观察方向的射线。

视图矩阵: 表示观察者的状态。
```
             [观察点位置]      (0,0,0)
viewMatrix = [观察点目标位置]  (0,0,-z)
             [上方向]          (0,1,0)
```
模型矩阵：图形经过旋转、位移、缩放变化得到的矩阵。

> 模型视图矩阵 = 视图矩阵 * 模型矩阵

可视空间（正射投影orthographic、透视投影perspective）/

投影矩阵。

> 模型视图投影矩阵 = 投影矩阵 * 模型视图矩阵

投影遮挡问题: 后绘制图形会覆盖先绘制图形。

解决：隐藏面消除

```
/*
 * @param gl.DEPTH_TEST  隐藏面消除
 *        gl.BLEND       混合
 *        gl.POLYGON_OFFSET_FILL 多边形位移
 *        ...
 */
gl.enable(gl.DEPTH_TEST); //开启某个功能

gl.disable()//关闭某个功能

gl.clear(gl.DEPTH_BUFFER_BIT); //清楚深度缓冲区

```
深度冲突

原因：当图形、物体的两个表面极为接近时，在深度缓冲区有限精度中分不清前后。

解决：webgl多边形偏移机制（在z轴上添加一个偏移量）
```
    gl.enable(gl.POLYGON_OFFSET_FILL);
    gl.polygonOffset(1.0, 1.0); //设置偏移量

```

- 创建方体

```
// 顶点示意图
//    v6----- v5
//   /|      /|
//  v1------v0|
//  | |     | |
//  | |v7---|-|v4
//  |/      |/
//  v2------v3
```
1、drawArrays()

（1）gl.TRIANGLES 绘制36个顶点（一面（两个三角形）6个顶点 * 6面）

（2）gl.TRIANGLES_FAN 绘制24个顶点，绘制6次。(一个面4个顶点 * 6面)

2、drawElements() 通过顶点索引来绘制

（1）gl.TRIANGLES 绘制8个顶点（共用顶点的关系）

（2）绘制不同面颜色时需要绘制24个顶点。

两者区别：

`bindBuffer()`和`bufferData()`时

drawArrays使用的是`gl.ARRAY_BUFFER`顶点数据

drawElements 使用的是`gl.ELEMENT_ARRAY_BUFFER`顶点索引

索引的存储使用`无符号整数型`存储，假设顶点数超过256个使用`Uint16Arrays()`来存储。

## ch08

着色的真正含义： 根据光照条件重建“物体各表面明暗不一的效果”。

着色的过程需要考虑：光源的类型、如何反射光线。

光源类型：

| 类型               | 现实中            | 反射颜色（无环境光）           | 反射颜色（有环境光）           | 入射角              |
| :--------------- | :------------- | :------------------- | -------------------- | ---------------- |
| 环境光（ambient）     | 非直射光（经过墙壁反射的光） | 入射光颜色 * 表面基底色        |                      |                  |
| 点光源（point）       | 类似灯泡光          | 入射光颜色 * 表面基底色 * cosθ | +  （入射环境光颜色 * 表面基底色） | 相同入射角            |
| 平行光（directional） | 类似太阳光          | 同上                   | 同上                   | 根据顶点计算各个不同的法线入射角 |
| 聚光灯（spot）        | 类似手电筒光         | 同上                   | 同上                   |                  |


> cosθ(入射角) = normalize（光线方向） ▪ normalize（法线方向）

> 矢量长度必须归一化处理（normalize），否则反射光颜色过暗过亮。

法线：

  1、垂直于表面的方向。

  2、一个面都具有两个法向量（正、反）。

  3、平面任意一点都具有相同的法向量。

- 当图形或物体有旋转或缩放时，法向量会产生变化。

解决：对模型矩阵进行逆矩阵。
```
//模型矩阵的逆矩阵，来变换法线向量矩阵
normalMatrix.setInverseOf(modelMatrix);

normalMatrix.transpose(); //将自身设为转置后的结果

...

在对法向量进行归一化时： normalize(vec3(u_normalMatrix * a_normal))
```

逐顶点计算与逐片计算的区别：前者会产生不自然的现象，原因：内插过程造成的。

> 点光时，光线方向的计算 光线方向 = normalize（光源世界坐标 - （顶点世界坐标 = 图形的模型矩阵 * 顶点位置））

