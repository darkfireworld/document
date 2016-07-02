# ListView嵌套ListView缓存失效问题

Android中在写列表的时候，相信很多时候，我们都需要进行ListView嵌套ListView编程。比如说：帖子+评论页面的编写。**然而这种模型是会出现被嵌套ListView缓存失效的问题。**

被嵌套的ListView的代码为：

	public class NestListView extends ListView {
		public NestListView(Context context, AttributeSet attrs) {
			super(context, attrs);
		}

		public NestListView(Context context) {
			super(context);
		}

		public NestListView(Context context, AttributeSet attrs, int defStyle) {
			super(context, attrs, defStyle);
		}

		@Override
		public void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
			//无限大小的子View空间
			int expandSpec = MeasureSpec.makeMeasureSpec(Integer.MAX_VALUE >> 2, MeasureSpec.AT_MOST);
			super.onMeasure(widthMeasureSpec, expandSpec);
		}
	}


最近，在写微信朋友圈的功能中，因为需要处理说说和评论，**所以采用ListView 嵌套ListView的处理模式。**而不幸的是，发生了评论的ListView的View缓存无法使用的情况，**从而造成GC比较频繁。**

	CommentAdapter adapter = (CommentAdapter) lv_lc_comment.getAdapter();
	if (adapter == null) {
		adapter = new CommentAdapter(context);
		lv_lc_comment.setAdapter(adapter);
	}
	//重新设置数据
	adapter.setData(cmTopicCommentList);
	adapter.setClsId(cmTopic.getClsId());
	adapter.setTag(tag);
	adapter.setOnClickListener(onTopicListener);
	adapter.notifyDataSetChanged();

上面这段代码就是评论adapter刷新代码。在实际运行过程中，会发生View Cache无法使用的问题。

调试代码，开始是怀疑不能多次调用setAdapter的问题**（之前都是没有添加adapter是否存在判断）**，而setAdapter代码为：

	public void setAdapter(ListAdapter adapter) {
		...
		mRecycler.clear();
		...
	}

可以发现，setAdapter（）会清空缓存。而采用最新的代码编写后，还是发生了View Cache无效的问题。再次翻阅ListView的代码**AbsListView# obtainView ：**

	View obtainView(int position, boolean[] isScrap) {
		...
		//查看是否存在相应的View缓存
		scrapView = mRecycler.getScrapView(position);
		...

		return child;
	}

可见，在ListView显示的时候，会去查询RecycleBin中是否存在对应的View缓存的，但是每次都是为null。

但是业务逻辑中，刷新一条说说的时候，会整体刷新它的评论，所以按照正常的逻辑来说，被嵌套的评论ListView中，应该包含了之前的评论View缓存。这一定是哪里出了问题。

所以看了一下**RecycleBin回收View的各个时机，**然后发现：

	protected void layoutChildren() {
		...
			//如果数据变化，则回收View
			if (dataChanged) {
				for (int i = 0; i < childCount; i++) {
					recycleBin.addScrapView(getChildAt(i), firstPosition+i);
					if (ViewDebug.TRACE_RECYCLER) {
						ViewDebug.trace(getChildAt(i),
								ViewDebug.RecyclerTraceType.MOVE_TO_SCRAP_HEAP, index, i);
					}
				}
			} else {
				recycleBin.fillActiveViews(childCount, firstPosition);
			}
		....
	}

**翻看了几个addScrapView几个被调用的地方后，发现layoutChildren这个调用最可疑，因为它利用dataChanged标志来判断是否回收View，而 notifyDataSetChanged就是标记需要回收View。**

并且**ListView#onMeasure->ListView#measureHeightOfChildren()** 需要进行获取子View高度的时候：

	final int measureHeightOfChildren(int widthMeasureSpec, int startPosition, int endPosition,

		...

		for (i = startPosition; i <= endPosition; ++i) {
			child = obtainView(i, isScrap);

			measureScrapChild(child, i, widthMeasureSpec);

			...
		}
		...
	}

**所以应该是NestListView这种无限长度的ListView的锅：**因为在onMeasure的时候obtainView所有的View高度，而RecycleBin是在onLayout的时候才能缓存不使用的View。而View#onMeasure() > View#onLayout()， 所以造成被嵌套的ListView缓存失效的问题。

**而最新的RecycleView已经可以支持View强制回收了API为：swapAdapter（）。**