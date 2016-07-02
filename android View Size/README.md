# Android View Size

在Android中，每个View具有它自己的View大小，而具体的属性就是：

* View#mLeft
* View#mTop
* View#mRight
* View#mBottom

这四个属性，具体大小设置依赖**ViewGroup#drawChild()**：

	protected boolean drawChild(Canvas canvas, View child, long drawingTime) {
		boolean more = false;
		//获取子View的空间大小
		final int cl = child.mLeft;
		final int ct = child.mTop;
		final int cr = child.mRight;
		final int cb = child.mBottom;
		//通知子View进行判断是否完成滚动，这里就是通过Scroller代码实现滚动的关键点
		child.computeScroll();
		//获取最新的偏移量
		final int sx = child.mScrollX;
		final int sy = child.mScrollY;
		//创建一个还原点
		final int restoreTo = canvas.save();
		//偏移，通过这个API，实现了scroll对内容偏移, 先把内容的原点进行偏移到负数区域
		canvas.translate(cl - sx, ct - sy);
		//剪切，因为之前有一个translate操作，所有剪切出来的空间就是父View给定的可见区域
		//所以如果子View填充Canvas的内容超出给定的空间，也不会显示出来
		canvas.clipRect(sx, sy, sx + (cr - cl), sy + (cb - ct));
		//让子View进行绘图，注意子View不用处理Scroll属性，既可以实现内容偏移
		child.draw(canvas);
		//还原
		canvas.restoreToCount(restoreTo);
		return more;
	}

可以发现，在ViewGroup#drawChild()的时候，通过**clipRect()**限制了子View的大小，即使子View高度/宽度超过父View也会被**截取**（排除ScrollView和ListView等特殊ViewGroup）。