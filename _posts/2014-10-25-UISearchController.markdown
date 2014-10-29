---
layout: post
title:  UISearchController
date:   2014-10-25 11:23:33
categories: iOS
---
UISearchController是iOS 8新添加的一个class，以来替代之前的UISearchDisplayController。

> The UISearchController class defines an interface that manages the presentation of a search bar in concert with the search results controller’s content. 

### SearchResultsController
在初始化`UISearchController`之前，需要先设计好它的*searchResultsController*。

*searchResultsController*用以管理和呈现搜索结果，是`UIViewController`类型的。通常设计为`UITableViewController`。

[JWSearchResultsController](https://github.com/Jowyer/JWSearchResultsController)是我做的一个基于`UITableViewController`类的*searchResultsController*，项目中还有一些使用示例。


### - initWithSearchResultsController:

通过这个方法，创建`UISearchController`。

{% highlight objc %}
JWSearchResultsController *searchResultsController = [self.storyboard instantiateViewControllerWithIdentifier:JWSearchResultsVCIdentifier];

self.searchController = [[UISearchController alloc] initWithSearchResultsController:searchResultsController];
{% endhighlight %}

### Launch Style
完成初始化后，下一步确定`UISearchController`将会在什么位置以何种形式被唤起。

1. Search Button

	![Search Button](/images/SearchController_SearchButton.png)

	通过点击搜索按钮，`UISearchController`出现在当前界面的上方。

	* {% highlight objc %}
	self.searchController.hidesNavigationBarDuringPresentation = NO;
[self presentViewController:self.searchController animated:YES completion:nil];
	{% endhighlight %}


2. 内嵌于TableView的顶部

	![TableView](/images/SearchController_TableView.png)

	点击搜索框，搜索框会滑到屏幕的顶部；或者搜索框在原有位置展开。
	
	* {% highlight objc %}
	//    self.searchController.hidesNavigationBarDuringPresentation = NO;
    self.tableView.tableHeaderView = self.searchController.searchBar;
    self.definesPresentationContext = YES;
    {% endhighlight %}
	
3. 内嵌于NavigationBar

	![Navigator](/images/SearchController_Navigator.png)
	
	搜索框直接位于导航栏的内部。
	
	注意：这种情况下就不要设置*searchBar.scopeButtonTitles*了，会因为显示区域太小而重叠在一起；*searchBar.searchBarStyle*也应该设置为*UISearchBarStyleMinimal*。

	* {% highlight objc %}
	//    self.searchController.searchBar.scopeButtonTitles = @[@"All", ProductTypeMobile, ProductTypeDesktop, ProductTypePortable];
self.searchController.searchBar.searchBarStyle = UISearchBarStyleMinimal;
self.searchController.hidesNavigationBarDuringPresentation = NO;
self.navigationItem.titleView = self.searchController.searchBar;
self.definesPresentationContext = YES;
    {% endhighlight %}


### delegate
search controller的代理，遵从`<UISearchControllerDelegate>`协议。主要是监听search controller显示逻辑方面的改变。

{% highlight objc %}
self.searchController.delegate = self;

#pragma mark - UISearchControllerDelegate
- (void)willPresentSearchController:(UISearchController *)searchController
{
    // Called when the search controller is to be automatically displayed.
}
{% endhighlight %}

###  searchResultsUpdater

`searchResultsUpdater`是一个属性，表示该updater负责更新搜索的结果，遵从`<UISearchResultsUpdating>`协议。

当搜索框成为*first responder*；框内的输入发生改变；搜索框的*scope*发生变化时，都会自动触发update方法。

{% highlight objc %}
self.searchController.searchResultsUpdater = self;

#pragma mark - UISearchResultsUpdating
-(void)updateSearchResultsForSearchController:(UISearchController *)searchController {
    /*
    This method is automatically called whenever the search bar becomes the first responder or changes are made to the text or scope of the search bar. 
    */
}
{% endhighlight %}

### searchBar 及 UISearchBarDelegate

`searchBar`属性是`UISearchController`内嵌的搜索栏，`UISearchBar`类型。

实现`<UISearchBarDelegate>`可以监听`searchBar`的变化。

{% highlight objc %}
self.searchController.searchBar.scopeButtonTitles = @[@"All", ProductTypeMobile, ProductTypeDesktop, ProductTypePortable];

self.searchController.searchBar.delegate = self;

#pragma mark - UISearchBarDelegate
- (void)searchBar:(UISearchBar *)searchBar selectedScopeButtonIndexDidChange:(NSInteger)selectedScope
{
    // Tells the delegate that the scope button selection changed.
}
{% endhighlight %}

### NSPredicate
搜索过程中一般使用`NSPredicate`。
        
{% highlight objc %}
NSPredicate *namePredicate = [NSPredicate predicateWithFormat:@"name CONTAINS [c] $nameString"];
[self.searchResults filterUsingPredicate:[namePredicate predicateWithSubstitutionVariables:@{@"nameString":searchString}]];
{% endhighlight %}











