# GDI MFC
based on this:
[MFC GDI基础知识](https://www.cnblogs.com/weiqubo/archive/2009/12/24/1930029.html)
## DC （device context）
设备描述表 = 画布 + 绘画工具
设备：显示器 打印机
实质：数据结构，存储输出时需要的信息。

+ 调用GDI函数绘制：
	首先 取得或建立设备环境句柄

## MFC中与GDI有关的类
两大类

1. 设备环境类：（CDC）封装了所有图形输出函数
	派生类：CClientDC, CPaintDC, WindowDC, CMetaFileDC
2. 绘图对象类：（CGDIObject）
	派生类CPen，CBrush，CFont，CBitmap，CPalette
	[MFC-GDI主要对象](https://blog.csdn.net/qwdpoiguw/article/details/72847925)
## GDI绘图步骤
### 1) 获取设备环境
（1）sdk编程：略
（2）mfc

∵ DC类都封装了DC句柄，构造时自动调用获取DC的API函数，析构时自动释放。
∴ 声明DC类对象即可自动获取DC。
OnDraw（）自动支持所获取的DC
```cpp
void CView::OnPaint()
{
         CPaintDC dc(this); // device context for painting
         // TODO: Add your message handler code here
         OnPrepareDC(&dc);
         OnDraw(&dc)
}
```

> 当我们改变了窗口尺寸、移动窗口或恢复了先前被覆盖的部分，应用程序窗口就会收到一个Windows系统发送来的WM_PAINT消息，然后调用基类Cview的OnPaint函数或我们自己添加的消息处理函数OnPaint。我们可以在OnPaint函数中重绘窗口中重新可见的部分，但简单的处理办法是重绘整个窗口。上面的代码中，由于基类Cview的OnPaint函数调用了OnDraw虚函数，因此应用程序经常在OnDraw函数中绘制视图。

### 2)设置坐标映射
### 3)创建绘图工具 + 选入DC
一般先创建画笔（or画刷），方法:

#### 1. 创建画笔
(1) Pen

```cpp
BOOL CPen::CreatePen( int nPenStyle, int nWidth, COLORREF cfColor );
```
```
nPenStyle 指定画笔的风格。其可能取值的列表，请参见CPen构造函数中的nPenStyle参数。
> nWidth 指定画笔的宽度。如果这个值为0，则不管是什么映射模式，以设备单位表示的宽度总是一个像素。 
> crColor 包含画笔的一个RGB颜色，为COLORREF结构。
```
>可通过`CDC::SelectStockObject`函数来调用系统预定义的库存笔对应的CGdiObject对象。
`pOldPen = (Cpen*)pDC->SelectStockObject(BLACK_PEN);`

（2）Brush
```cpp
BOOL CBrush::CreateSolidBrush ( COLORREF crColor );
 
BOOL CBrush::CreateHatchBrush( int nIndex, COLORREF crColor );
```
参数： nIndex 指定画刷的阴影线风格。可取的值如下：
```
HS_HORIZONTAL   /* ==== */
HS_VERTICAL    /* ||||| */
HS_FDIAGONAL /* \\\\\ */
HS_BDIAGONAL /* ///// */
HS_CROSS       /* +++++ *
HS_DIAGCROSS /* xxxxx */
```
返回值：调用成功时返回非零值，否则为0。
此外，可通过CDC::SelectStockObject函数来调用系统预定义的库存画刷对应的CGdiObject对象。
`pOldBrush = (CBrush*)pDC->SelectStockObject(BLACK_BRUSH);`


#### 2. 调用`CDC::SelectObject` 将其选入设备环境，作为当前绘图工具。

以下为MFC中默认映射方式下的GDI绘图的模块：
 ```cpp
	/**先获取设备环境pDC**/
    CPen *pOldPen,newPen;
    CBrush *pOldBrush,newBrush1,newBrush2;

    //创建宽度为pixel的白色实线画笔
    newPen.CreatePen(PS_SOLID, 1, RGB(0, 0, 0));
    //创建红色实线画刷
    newBrush1.CreateSolidBrush(RGB(255,0,0));
    //创建红色实线度的向下（从右到左）影线的阴影画刷
    newBrush2.CreateHatchBrush(HS_BDIAGONAL,RGB(255,0,0));

    /*将newPen画笔和newBrush1画刷对象选入设备环境*/
    pOldPen = pDC->SelectObject(&newPen);
    pOldBrush = pDC->SelectObject(&newBrush1);
 
    //调用DC绘图函数绘图
 
    //……
 
    //绘图完毕，恢复原来画笔、画刷
    pDC->SelectObject(pOldPen);
	pDC->SelectObject(pOldBrush);
 
	//删除创建的画笔、画刷
 
	// newPen.DeleteObject();
 
	// newBrush1.DeleteObject();
 
	// newBrush2.DeleteObject();
```
#### 3) 绘图完毕，回复以前的画笔
#### 4) 调用`CDC::DeleteObject`删除对象
> 在MFC中，当对象销毁时会调用对象的析构函数自动删除对象，一般不必调用CGdiObject::DeleteObject删除GDI对象，因为如果设备环境还在使用一个GDI对象时，将引起应用程序崩溃或出现难以理解的运行错误。

## 绘图函数
![GDI function](https://images.cnblogs.com/cnblogs_com/weiqubo/2.PNG)


