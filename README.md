blog地址：http://tangpj.com
# Android SearchView的高级用法，解决关于SearchView的样式与控制问题
在Android开发的时候，有时候我们需要做一个关于搜索的模块。我发现很多初级工程师在开发搜索组件的时候会用EditText + Button + ListView(RecyclerView) 的方法重新制作一个searchView组件。虽然这种方案也是可行的，但是效果往往不尽人意。现在我来介绍种更为简单的方法，就是实用android.widget包下的SearchView控件。

# 关于SearchView
## 什么是SearchView控件？
SearchView是Android自带的一款搜索View，它的功能十分强大，而且样式也十分好看。在Google发布Material Design 介绍中的搜索按钮就是通过SearchView实现的。

## 如何使用SearchView
下面我们来介绍下在Material Design风格下如何使用SearchView。
1. 在toolbar中显示SearchView。
在res/menu/目录下创建文件menu_main.xml
<menu xmlns:android="http://schemas.android.com/apk/res/android"
xmlns:app="http://schemas.android.com/apk/res-auto"
xmlns:tools="http://schemas.android.com/tools"
tools:context="com.example.searchview.MainActivity">

<item android:id="@+id/search_view"
android:orderInCategory="101"
android:title="search"
android:icon="@mipmap/search"
app:actionViewClass="android.widget.SearchView"
app:showAsAction="ifRoom|collapseActionView"
/>
</menu>



配置完菜单选项后还需要在使用的Acitvity中添加如下代码，使用配置好的菜单
@Override
public boolean onCreateOptionsMenu(Menu menu) {
// Inflate the menu; this adds items to the action bar if it is present.
getMenuInflater().inflate(R.menu.menu_main, menu);
return true;
}

从上面代码可以看出，SearchView的使用和一般的菜单项没有太大的区别。只是多了app:actionViewClass这句代码。app:actionViewClass的作用是指定菜单项的行为视图，它的作用是点击菜单项后会自动根据点击时间显示相应的View组件。所以在这里可以推断，除了设置为SearchView外，还可以把这个View设置为任何你想设置的View（Button,EditText、自定义View等），在这里先不展开讨论。

![](https://raw.githubusercontent.com/DobbyTang/MarkdownRes/master/mBlog/searchView/2016-09-10%2013_52_48.gif)

2. 响应SearchView的的行为
设置好SearchView后，我们要对SearchView进行监听，监听的方法普通使用SearchView一样。获取SearchView对象并进行监听的方法如下所示。
@Override
public boolean onCreateOptionsMenu(Menu menu) {
// Inflate the menu; this adds items to the action bar if it is present.
getMenuInflater().inflate(R.menu.menu_main, menu);
final MenuItem item = menu.findItem(R.id.search_view);
SearchView searchView = (SearchView) item.getActionView();
searchView.setOnQueryTextListener(new     SearchView.OnQueryTextListener() {
@Override
public boolean onQueryTextSubmit(String query) {
Toast.makeText(MainActivity.this,query,Toast.LENGTH_SHORT).show();
return false;
}

@Override
public boolean onQueryTextChange(String newText) {
return false;
}
});
return true;
}

首先，通过Menu.findItem（int id）方法，通过我们在`menu_main.xml `文件中定义的id获取到对应的menuItem,然后在通过MenuItem.getActionView()方法获取actionViewClass(响应行为View，在本例子里就是获取对应的SearchView)。获取到ActionView的实例后，我们就可以通过调用对象的一些方法实现相应的行为了。例如在这里就是用Toast把输入的搜索词语显示到屏幕上。

![](https://raw.githubusercontent.com/DobbyTang/MarkdownRes/master/mBlog/searchView/2016-09-11%2015_56_45.gif)

# 自定义SearchView的样式与重设监听事件
上面已经介绍了SearchView的简单的用法了，但是往往在实际开发的过程中并不能满足我们的需求。SearchView实际上是一款十分强大的View控件来的，下面我们来介绍下它到底有多么强大。
## SearchView源码分析
首先我们来看看它的源码。
private static final boolean DBG = false;
private static final String LOG_TAG = "SearchView";

/**
* Private constant for removing the microphone in the keyboard.
*/
private static final String IME_OPTION_NO_MICROPHONE = "nm";

private final SearchAutoComplete mSearchSrcTextView;
private final View mSearchEditFrame;
private final View mSearchPlate;
private final View mSubmitArea;
private final ImageView mSearchButton;
private final ImageView mGoButton;
private final ImageView mCloseButton;
private final ImageView mVoiceButton;
private final View mDropDownAnchor;

我们发现SearchView实际上是一个View的集合，有4个View、ImageView,还有一为SearchAutoComplete。我们重点关注的就是这个叫SearchAutoComplete的控件。后面大多数的功能都是围绕这个控件展开的。

### SearchAutoComplete控件是什么？
我们对SearchAutoComplete的源码进行查看，简单分析它的功能。
`public static class SearchAutoComplete extends AppCompatAutoCompleteTextView `
通过查看源码我们可以知道SearchAutoComplete实际上就是一个AutoCompleteTextView,而AutoCompleteTextView。所以SearchAutoComplete实际上是对AutoCompleteTextView的一个拓展。而我们知道SearchView是有一个输入框和多个按钮的。所以我判断，这个SearchAutoComplete实际上就是SearchView上的输入框，那么我们可以根据这个特点，完成一些AutoCompleteTextView类似的功能。

### SearchAutoComplete控件能做什么？
SearchAutoComplete的特点就是有自动补全的功能。那么我们可以通过这个功能做一个类似自动补全的功能，实际上很多Android中的自动补全功能都是通过AutoCompleteTextView来实现的。所以我们可以通过SearchAutoComplete的这个特性来实现搜索提示或者历史记录功能都可以。

## SearchView搜索提示（历史记录）的实现
先让大家看看实现搜索提示的代码

自定义搜索提示数组：
<?xml version="1.0" encoding="utf-8"?>
<resources>
<string-array name="test_array">
<item>香港</item>
<item>广州</item>
<item>深圳</item>
<item>北京</item>
<item>上海</item>
<item>杭州</item>
<item>天津</item>
<item>长沙</item>
</string-array>
</resources>


实现搜索提示代码：
@Override
public boolean onCreateOptionsMenu(Menu menu) {
// Inflate the menu; this adds items to the action bar if it is present.
getMenuInflater().inflate(R.menu.menu_main, menu);
Resources resources = getResources();
String [] testStrings = resources.getStringArray(R.array.test_array);
final MenuItem item = menu.findItem(R.id.search_view);
SearchView searchView = (SearchView) item.getActionView();

int completeTextId = searchView.getResources().getIdentifier("android:id/search_src_text", null, null);
AutoCompleteTextView completeText = (AutoCompleteTextView) searchView
.findViewById(completeTextId) ;
completeText.setThreshold(0);
completeText.setAdapter(new ArrayAdapter<>(this,android.R.layout.simple_list_item_1,testStrings));
searchView.setOnQueryTextListener(new SearchView.OnQueryTextListener() {
@Override
public boolean onQueryTextSubmit(String query) {
Toast.makeText(MainActivity.this,query,Toast.LENGTH_SHORT).show();
return false;
}

@Override
public boolean onQueryTextChange(String newText) {
return false;
}
});

return true;
}

从代码可以看出来
1. AutoCompleteTextView ID的获取方法为`int completeTextId = searchView.getResources().getIdentifier("android:id/search_src_text", null, null);`。为什么这样能获取到searchView中的AutoCompleteTextView的ID呢？我们先来看看SearchView的源码：
private final SearchAutoComplete mSearchSrcTextView;
mSearchSrcTextView = (SearchAutoComplete) findViewById(R.id.search_src_text);

我们可以看到，在SearchView中SearchAutoComplete的id是`R.id.search_src_text `所以我们可以通过View.getResources(). getIdentifier()方法来获取View的内部控件（该方法对所有的View都适用）。

2. completeText.setThreshold(0)方法的作用是设置提示条件。设置为0的时候就是代表输入长度为0即显示提示列表，设置为1代表输入长度为1时显示提示列表，以此类推。

3. 通过completeText.setAdapter()方法设置搜索提示的数据源。搜索提示View实际上是一个ListView所以这里的setAdapter方法和ListView是一样的。为了演示方便，在这里我使用了ArrayAdapter。

现在我们来运行看看显示的效果吧。

![](https://raw.githubusercontent.com/DobbyTang/MarkdownRes/master/mBlog/searchView/2016-09-11%2021_09_12.gif)

嗯，符合我们的预期，但是在我们点击下面的搜索提示的时候，程序蹦贵了。报崩溃的原因是因为SearchView搜索结果在内部是通过Cursor传递的。我们通过查看源码可以看到，SearchView的搜索结果是通过SuggestionsAdapter显示的，它的数据是来源是Cursor。而在我们的源码中是通过ArrayAdapter实现的，所以造成数据不一致所以报错了。我们知道在android中Cursor是用来操作查询数据集合的，但是实际开发中，我们的数据源不一定是来源于数据库。所以在这里我们需要对SearchView进行改造下，使它满足我们的使用目的。

解决的方法为:
通过上面的分析，我们知道点击搜索提示列表中的搜索结果时实际上是在点击AutoCompleteTextView中的ListView的item。而通过查看AutoCompleteTextView 的源码我发现又一个setOnItemClickListener方法。通过这个方法重新设置OnItemClickListener即可解决这个问题。下面来看看解诀后的代码。
@Override
public boolean onCreateOptionsMenu(Menu menu) {
// Inflate the menu; this adds items to the action bar if it is present.
getMenuInflater().inflate(R.menu.menu_main, menu);
Resources resources = getResources();
final String [] testStrings = resources.getStringArray(R.array.test_array);
final MenuItem item = menu.findItem(R.id.search_view);
final SearchView searchView = (SearchView) item.getActionView();

int completeTextId = searchView.getResources().getIdentifier("android:id/search_src_text", null, null);
AutoCompleteTextView completeText = (AutoCompleteTextView) searchView
.findViewById(completeTextId) ;
completeText.setAdapter(new ArrayAdapter<>(this,R.layout.list_item,R.id.text,testStrings));
completeText.setOnItemClickListener(new AdapterView.OnItemClickListener() {
@Override
public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
searchView.setQuery(testStrings[position],true);
}
});

completeText.setThreshold(0);
searchView.setOnQueryTextListener(new SearchView.OnQueryTextListener() {
@Override
public boolean onQueryTextSubmit(String query) {
Toast.makeText(MainActivity.this,query,Toast.LENGTH_SHORT).show();
return false;
}

@Override
public boolean onQueryTextChange(String newText) {
return false;
}
});


return true;
}


现在我们来分析下`searchView.setQuery(testStrings[position],true); `的这个方法。下面时这个方法的定义:

public void setQuery(CharSequence query, boolean submit) {
mSearchSrcTextView.setText(query);
if (query != null) {
mSearchSrcTextView.setSelection(mSearchSrcTextView.length());
mUserQuery = query;
}

// If the query is not empty and submit is requested, submit the query
if (submit && !TextUtils.isEmpty(query)) {
onSubmitQuery();
}
}

不难看出，query就是代表设置要提交的字符串，而submit为ture表示提交该字符串，为false为不提交。这样我们就可以达到该方法实现选择搜索提示的效果。根据这一特性，我们可以在实际开发的时候根据业务逻辑自定义Adapter来实现各种复杂的功能。
demo地址为： https://github.com/DobbyTang/SearchViewDemo

# 小结
SearchView是Android提供的一个十分强大的搜索组件，在这里我只是自定义了其中的一小部分功能。我曾经听到过很多小伙伴抱怨SearchView十分不好用，一般在开发搜索功能的时候都是重新制作一个搜索模块。但是通过对SearchView的源码进行分析可以发现，自定义该控件的方法实际上十分简单。并且随着对它的深入分析发现，一个完整的搜索模块需要的东西它都以更加优美的方法帮我们实现了。在开发的时候，我们不妨尝试下去使用这个十分强大的控件。
通过这篇文章，我想告诉大家的是，多看源码！！！多看源码！！！多看源码！！！看源码的过程很辛苦，但是这是我们获得进步的最快途径，并且能在看源码的过程中能够发现一些控件的十分强大的用法。虽然不一定能完全理解，但是当你使用源码中的一些方法做出一些很酷的事情的时候，你会发现有满满的成就感，SO,有空的时候看看源码吧。



