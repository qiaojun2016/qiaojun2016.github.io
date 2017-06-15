---
layout: post
date: 2017-03-01
title: 一个Inflate引发的思考
---
### 背景
今天项目中需要使用RecycleView，但是我差不多已经忘了该如何使用了。作为代码的搬运工，我迅速从网上找到了模板，三两下把代码给折腾完了。然后我自己根据需求写了个简单的item.xml 大概这么写的
{% highlight xml %}
```
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="96dp">
    <ImageView
        android:layout_centerInParent="true"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:src="@drawable/icon_camera"
        />
</RelativeLayout>


 <RecyclerView
        android:id="@+id/recycler"
        android:layout_width="match_parent"
        android:layout_height="96dp">
</RecyclerView>
```
{% endhighlight %}
在onCreateViewHolder中（我使用的是LinearLayoutManager，横向布局）:
```
    View view = LayoutInflater.from(context).inflate(R.layout.item,null);
```
但是问题出现了，假如你也实验了，你就会发现每个item都没有垂直居中显示，我的意思是说，按照我当前的理解，因为我的RecyclerView刚刚设置了高度**96dp**，而我的item的高度也是**96dp**，那么我的ImageView又设置了**layout_centerInParent = true**属性，根据正常逻辑，必须垂直居中的。后来我把item的高度改为**match_parent**,然而并没有什么卵用，所以我就怀疑item的这个 **RelativeLayout** 的高度和宽度根本就没有用到。
那么问题出在哪呢？我直接说出来大家看看这个是不是有问题：
```
View view = LayoutInflater.from(context).inflate(R.layout.item, null);
```
```
View view = LayoutInflater.from(context).inflate(R.layout.item,parent, false);
```
首先这个parent我用的是 **onCreateViewHolder** 传递过来的参数。这个 **parent**参数的含义，相比大家都知道了。如果不知道，那我告诉你先(贴个源码)：
```
   public View inflate(XmlPullParser parser, @Nullable ViewGroup root, boolean attachToRoot) {
        synchronized (mConstructorArgs) {
            Trace.traceBegin(Trace.TRACE_TAG_VIEW, "inflate");

            final Context inflaterContext = mContext;
            final AttributeSet attrs = Xml.asAttributeSet(parser);
            Context lastContext = (Context) mConstructorArgs[0];
            mConstructorArgs[0] = inflaterContext;
            View result = root;

            try {
                // Look for the root node.
                int type;
                while ((type = parser.next()) != XmlPullParser.START_TAG &&
                        type != XmlPullParser.END_DOCUMENT) {
                    // Empty
                }

                if (type != XmlPullParser.START_TAG) {
                    throw new InflateException(parser.getPositionDescription()
                            + ": No start tag found!");
                }

                final String name = parser.getName();
                
                if (DEBUG) {
                    System.out.println("**************************");
                    System.out.println("Creating root view: "
                            + name);
                    System.out.println("**************************");
                }

                if (TAG_MERGE.equals(name)) {
                    if (root == null || !attachToRoot) {
                        throw new InflateException("<merge /> can be used only with a valid "
                                + "ViewGroup root and attachToRoot=true");
                    }

                    rInflate(parser, root, inflaterContext, attrs, false);
                } else {
                    // Temp is the root view that was found in the xml
                    final View temp = createViewFromTag(root, name, inflaterContext, attrs);

                    ViewGroup.LayoutParams params = null;

                    if (root != null) {
                        if (DEBUG) {
                            System.out.println("Creating params from root: " +
                                    root);
                        }
                        // Create layout params that match root, if supplied
                        params = root.generateLayoutParams(attrs);
                        if (!attachToRoot) {
                            // Set the layout params for temp if we are not
                            // attaching. (If we are, we use addView, below)
                            temp.setLayoutParams(params);
                        }
                    }

                    if (DEBUG) {
                        System.out.println("-----> start inflating children");
                    }

                    // Inflate all children under temp against its context.
                    rInflateChildren(parser, temp, attrs, true);

                    if (DEBUG) {
                        System.out.println("-----> done inflating children");
                    }

                    // We are supposed to attach all the views we found (int temp)
                    // to root. Do that now.
                    if (root != null && attachToRoot) {
                        root.addView(temp, params);
                    }

                    // Decide whether to return the root that was passed in or the
                    // top view found in xml.
                    if (root == null || !attachToRoot) {
                        result = temp;
                    }
                }

            } catch (XmlPullParserException e) {
                final InflateException ie = new InflateException(e.getMessage(), e);
                ie.setStackTrace(EMPTY_STACK_TRACE);
                throw ie;
            } catch (Exception e) {
                final InflateException ie = new InflateException(parser.getPositionDescription()
                        + ": " + e.getMessage(), e);
                ie.setStackTrace(EMPTY_STACK_TRACE);
                throw ie;
            } finally {
                // Don't retain static reference on context.
                mConstructorArgs[0] = lastContext;
                mConstructorArgs[1] = null;

                Trace.traceEnd(Trace.TRACE_TAG_VIEW);
            }

            return result;
        }
    }
```
源码也算不多。

于是，现在大家想一下这样一个问题，我的 **ImageView** 它的 **layout_height** 和 **layout_width**是给谁使用的？都知道是用来设置 **ImageView** 的高度和宽度的。所以我对这俩属性的定义就是：**"这俩属性是用来设置其在父控件中的高度和宽度的，如果没有了父控件，那么这些高度和宽度将没有意义"**。在属性的命名上不知发现了没有，为什么是 **layout_** 开头的呢，而不是**android:width** 明显这是有意义，说明是与父控件相关的，或者说父控件需要这个值。

回到原来的问题bug，也就是inflate的问题，我简单的看了一下 **inflate** 的源码，基本过程如下
>1. 创建视图的根布局(这里就是 **RelativeLayout** )。
>2. 如果 **parent** 是存在的，就用 **parent** 返回一个 **LayoutParams** ,给这个根布局(**RelativeLayout**)使用，不存在就不会设置。并且 **attachToRoot** 为 **true** 那么就把根布局添加到 **parent**，否则就不添加。
>3. 递归创建 **childView**。

其实，简单说来就是解析我们的item.xml文件，动态递归的从上到下创建这个View。
想必同学们都用过动态创建过View，比如动态创建一个 **TextView**，需要创建后添加到 **LinearLayout** 中，如果你没有主动调用 **TextView.setLayoutParams(..)**,你在**addView**的时候，**LinearLayout** 会主动为你加上。如果不加上，LinearLayout就不知道这个**TextView**需要占多大的位置。

回到项目中遇到的问题：因为我是用下面的方法填充的:
```
LayoutInflater.from(context).inflate(R.layout.item, null)
```
结果导致了并没有设置生成的**RelativeLayout**的**LayoutParams**,然后在向**RecyclerView**添加过程中，**RecyclerView**发现，要被添加的View竟然没有**LayoutParams**参数，于是**RecyclerView**就默认为**RelativeLayout**设置了两个**wrap_content**。于是item布局发生了这样变化：
```
andrid:layout_width = "match_parent"
android:layout_height = "96dp"
```
变成了
```
andrid:layout_width = "wrap_content"
android:layout_height = "wrap_content"
```

至于怎么解决，看你们的了。
