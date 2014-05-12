---
layout: post
title: "Rewriting AngularJS app in React - part I"
date: 2014-05-12 23:55
tags: [angularjs, react, javascript]
---
AngularJS is great but it has some [confusing parts](http://codeofrob.com/entries/you-have-ruined-javascript.html). On the other hand [React](http://facebook.github.io/react) is getting more and more attention. 

Talking to [Hadi](http://twitter.com/hhariri) we agreed that we have to rewrite our AngularJS applications to React as it's the latest and shinest of all things apparently.

---
### TL;DR

I'll be rewriting specific AngularJS app to React just for the **sake of learning**.

---

Started as a joke but I decided to see what React is all about and implement some of functionalities I already developed together with [rafek](http://twitter.com/rafek) in AngularJS. I'll continue doing this till the point I feel satisfied with my understanding of React. It's not my goal to rewrite the whole application but to get the feeling what React is all about.

It's not meant to be a tutorial. React creators have that [covered already](http://facebook.github.io/react/docs/tutorial.html). It's meant to be my approach to learn React on specific example trying to reimplement app already written in AngularJS. I'll be happy to share what I've learned but I don't guarantee it'll be the best way to learn React nor I can assure that I'll cover most important parts of React here.

First of all React is not yet-another-Model-View-Whatever framework. It was built by Facebook engineers to [build large applications with data that changes over time](http://facebook.github.io/react/docs/why-react.html) and its core is View. You build something called virtual DOM (yeah, you have not misheard that) and from that React renders actual html document every time you change your data. Sounds like a crazy idea but it's actually smart enough to know when to throw away the whole document and reconstruct it from scratch and when apply only diff changes to it.

### Application I'm reimplementing 
It's some kind of gantt-chart which allows you to have tree structure. Each node is either grouping other elements or have activities that can span across certain time. Looks like this so far:
![app I'm rewriting](/images/app_i_am_rewriting.png)

Important thing is that each operation on a tree structure is shown on the right which is our main view. Each operation on the main view is also reflected in the tree on the left. On the left you see folders and instances (if we don't have instance in the *selected* year - there's an \<empty\> tag). On the main view we see instances (activities in a gantt-chart) which can span over several years. Whenever you select a year - tree structure on the left shows you representation of itself for the selected year.

### First component
Let's start with building first component. It'll contain the whole application. I'll call it LCC as it's the original name of our app.

{% highlight html linenos %}
<body>
	<div id="container">
	</div>
	<script src="/lib/react.js"></script>
	<script src="/lib/JSXTransformer.js"></script>
	<script type="text/jsx" src="lcc.js"></script>
</body>
{% endhighlight %}

I linked two scripts need for React - react.js and JSXTransformer.js. The last one is my application I'm going to write using React.

---
*Note:* React uses JSXTransformer to translate from virtual DOM represented in XML (sounds scary, huh?) to actual Javascript objects. All of the below can be done without using JSX which can scary some people.

---

So let's look at lcc.js:
{% highlight javascript linenos %}
/**
 * @jsx React.DOM
 */
{% endhighlight %}

It starts with declaration for JSXTransformer to treat underlying text as potentially JSX code to translate. 

Here is the first component:
{% highlight javascript linenos %}
var LCC = React.createClass({
  render: function() {
    return (
		<div>
			<div id="left-sidebar">
				<ProductTree />
			</div>
			<div id="main-content">
				<Roadmap />
			</div>
		</div>
	);
  }
});

React.renderComponent(<LCC />, document.getElementById('container'));
{% endhighlight %}

Let's look from the bottom. In line 16 I'm rendering LCC component in my 'container' element in html. That will replace the whole div with what I specify as LCC. Creating component starts at the first line. It uses only 'render' function to, guess what, render its contents. And here comes JSX - it's XML notation to describe how my component should render. It contains another two components - 'ProductTree' and 'Roadmap'. It represents what I named previously tree and main view. Their implementation looks like that:

{% highlight javascript linenos %}
var ProductTree = React.createClass({
	render: function() {
		return (<div>tree</div>);
	}
});
var Roadmap = React.createClass({
	render: function() {
		return (<div>roadmap</div>);
	}
});
{% endhighlight %}

They render one simple div each.
Running this in the browser will give me:
![start](/images/start.png)

---
*Note:* I told you that you could achieve the same without JSX - it's meant to be just syntactic sugar. Each component could be declared this way:
{% highlight javascript linenos %}
var Roadmap = React.DOM.div({}, 'roadmap');
{% endhighlight %}
I'll be using JSX though just to see if it hurts me in time.

---
### Passing data to component


My testing data structure looks like this (well, it's a bit more complicated but I'll try to minimize it for the sake of understanding):

{% highlight javascript linenos %}
var Data = {
	tree: [
		{
			"label" : "System", "children" : [
				{
					"label" : "Folder", children : [
						{
							"current" : { "label" : "Instance 1" }
						},
						{
							"label" : "Folder 2", children : []
						}
					]
				},
				{
					"current" : null
				},
				{
					"current" : null
				}
			]
		}
	]
};
{% endhighlight %}

Nothing so fancy about it, tree structure with folders and what we call - instances. Some of them don't have 'current' item - this is because they don't have any activity this year, let's leave it for now.

I want to pass this data to my ProductTree component. I'm doing this through "props" which is analogical to element attributes in html.
{% highlight html linenos %}
<ProductTree data={Data} />
{% endhighlight %}

I'll change definition of ProductTree to be using newly passed data:
{% highlight javascript linenos %}
var ProductTree = React.createClass({
	render: function() {
		return (
			<div>
				<ul>
					<ProductTreeFolder folder={this.props.data.tree[0]} />
				</ul>
			</div>
		);
	}
});
{% endhighlight %}
I'm passing `this.props.data.tree[0]` using `folder` attribute to newly created component - ProductTreeFolder. I had my previously passed `data` in `this.props` object which is available for every component. Inside `data` I'm going to `tree` and grab its first element. Easy.

Let's look at ProductTreeFolder implementation:
{% highlight javascript linenos %}
var ProductTreeFolder = React.createClass({
	render: function() {
		var children = [];
		for(var i = 0; i < this.props.folder.children.length; i++) {
			if (!this.props.folder.children[i].children) {
				children.push(<ProductTreeNode node={this.props.folder.children[i]}/>);
			}
			else {
				children.push(<ProductTreeFolder folder={this.props.folder.children[i]}/>);
			}
		}

		return (
		<li>
			{this.props.folder.label}
			<ul>{children}</ul>
		</li>
		);
	}
});
{% endhighlight %}
This function will go through all children and render either ProductTreeFolder recurrently or ProductTreeNode with `node` passed as another 'prop'. After finishing this it renders itself as \<li\> tag with folder label. Easy?

ProductTreeNode is even simpler:
{% highlight javascript linenos %}
var ProductTreeNode = React.createClass({
	render: function() {
		if (this.props.node.current) {
			return (
			<li>
				{this.props.node.current.label}
			</li>
			);
		}
		else {
			return (
			<li>
				empty
			</li>
			);
		}
	}
});
{% endhighlight %}
It renders itself as \<li\> with either current label or empty.

After running this we have a proper tree structure in our DOM:
![first results](/images/first_results.png)

### Enough for now, right?
Next time I'll try to make both sides of my app rendering html from one data source. 

So far I've learned:

  * how to create component in React
  * how to pass data to it
  * what JSX is
  * how to see what virtual DOM React has created for me - thanks to [this chrome extension](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi)
  * probably something more not described in this post 😃

---
Code of all this is available on [GitHub](http://github.com/mihcall/lcc-react.git). It may differ from snippets from the blog as I was playing with some styling and minimized data structures for the sake of understanding on this blog.


---
*Note:* I know that React is not a silver bullet and it doesn't solve the problem of [Javascript being ruined](http://codeofrob.com/entries/you-have-ruined-javascript.html) but I was somehow encouraged to look at it anyway by several people including [Dan North](http://twitter.com/tastapod) so - why not. Kill me but I like the idea of trying things even if they contribute to destroying nice hip languages. Anyway, Javascript is already mainstream, hipsters do not care anymore.
