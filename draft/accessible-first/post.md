# Accessible-first

Building an accessible product sounds so obvious, yet it is very often overlooked or perceived as a separate entity of the product. I, myself, have fallen victim to this frame of thought. Only after having worked on several accessibility-related projects did version 3.4 of [pickadate.js](http://amsul.ca/pickadate.js) come out with some ARIA controls. Not implementing it sooner was mostly due to a lack of time, but partly also due to me expecting it to be a daunting task.

I’d like to stress here [how](http://www.w3.org/WAI/intro/accessibility.php#important) [crucial](http://webaccessibility.jhu.edu/what-is-accessibility/important.html) [accessibility](http://www.visionaustralia.org/business-and-professionals/digital-access/why-accessibility-is-important) [on the web](http://eip.reach.ca/default/main/products/service-provider-s-companion/english/the-importance-of-accessibility.html) has become. If your product isn’t accessible, it means that it doesn’t *really* fully function. Yes, you might have customers coming to your site - but a large population of visitors will not even be given the chance to be your customer, simply because they cannot access it!

It’s like architecting a building without planning for elevators.

It leaves a lost opportunity by not being inclusive. And heck, it’s even [the law now in several countries](http://www.aodacompliance.com/). So I think we ought to spend a little time learning the basics of building an accessible application.


## All about the semantics

HTML (specifically HTML5) tries to inherently be semantic enough to kick start an accessible application. Screen readers know how to navigate and prioritize content based on your markup flow:

```html
<body>
	<header></header>
	<aside></aside>
	<main>
		<header></header>
		<article></article>
		<footer></footer>
	</main>
	<footer></footer>
</body>
```

However, HTML semantics alone cannot appropriately describe all possible content types in an application. And that’s when [ARIA](http://www.w3.org/TR/wai-aria/) controls come in.


## ARIA (Accessible Rich Internet Applications)

ARIA controls give us the ability to define our own semantics where HTML falls short. These definitions are categorized into four [roles](http://www.w3.org/TR/wai-aria/roles#roles_categorization):

1. **[Landmark roles](http://www.w3.org/TR/wai-aria/roles#landmark_roles)** define the general regions within an application that become “navigational landmarks” for screen readers - sort of like “Jump to”s.
2. **[Document structure roles](http://www.w3.org/TR/wai-aria/roles#document_structure_roles)** add semantics to content organization in a page and are usually not interactive.
3. **[Widget roles](http://www.w3.org/TR/wai-aria/roles#widget_roles)** describe components of UI widgets.
4. **[Abstract roles](http://www.w3.org/TR/wai-aria/roles#abstract_roles)** just define the ontology of other role types. These are *not* to be used in an application.

The spec documentation is a bit dry to read through, but essentially, there are three basic steps to building an accessible application:

1. Start by architecting the **landmark** roles,
2. Then build out the **document structure** roles, and
3. Finally, create the individual **widget** roles.

There’s a [long list of roles](http://www.w3.org/TR/wai-aria/roles#role_definitions) that fall under these categories. We will be looking at the most fun part (widget roles) and use a date picker as an example UI component.


### Widget roles

There are two types of widget roles:

1. Composite roles, and
2. Standalone roles.

A composite role defines a “container” for the widget’s UI components whereas the standalone roles are components within the UI’s container.

So, let’s say the basic interaction for the date picker we want is that the user can choose a date from a grid of choices. The following is our markup to start with (for brevity, I’m ignoring irrelevant HTML attributes):

```html
<input>
<table>
	...
	<tr>
		...
		<td>19</td>
		<td>20</td>
		...
	</tr>
	...
</table>
```

A screen reader visiting this markup sees two disparate pieces of DOM elements. One an `input` that semantically accepts a value, and the other a `table` that semantically displays tabular data.

Already, we have a problem with those semantics. We aren’t using this `table` for tabular data. We’re using it to visually present the content in a grid layout - which is just easily achieved with the native `table` element.

In order to correct this, we need to redefine the role of our `table`.

It is crucial to note here that most semantic HTML elements cannot have their roles redefined. Only [certain elements can be redefined certain other roles](http://rawgithub.com/w3c/aria-in-html/master/index.html#rec). For the most part, `div`s and `span`s can be redefined to mean anything.

Anyways, from the [long list of roles](http://www.w3.org/TR/wai-aria/roles#role_definitions), the most appropriate role for our scenario is the [composite `grid` role](http://www.w3.org/TR/wai-aria/roles#grid). The `grid` role requires that there be at least one element with the role of `gridcell` contained by an element with the role of `row`, which in turn is owned by the `grid`.

Essentially that means the following roles need to be in our markup:

```html
<input>
<table role="grid">
	<tr role="row">
		...
		<td role="gridcell">19</td>
		<td role="gridcell">20</td>
		...
	</tr>
</table>
```

When a screen reader comes across the markup above, the following accessibility tree is generated:

```html
<input></input>
<grid>
	<row>
		...
		<gridcell></gridcell>
		<gridcell></gridcell>
		...
	</row>
</grid>
```

... redefining the semantic meaning of the `table` container and content within.

There’s [many more roles](http://www.w3.org/TR/wai-aria/roles#role_definitions) that could fit this scenario - it all depends on the interaction behavior we want the widget to have. An important role to be aware of is [`presentation`](http://www.w3.org/TR/wai-aria/roles#presentation). This role removes all semantic meaning from an element:

```html
<h1 role="presentation">I will be semantic-less</h1>
<span>I am inherently without any semantics</span>
<ul role="presentation">
	<li>I have no semantics either</li>
	<li>But I have <b>some</b></li>
</ul>
```

When the screen reader sees this markup, the accessibility tree looks like this:

```html
<>I will be semantic-less</>
<>I am inherently without any semantics</>
<>
	<>I have no semantics either</>
	<>But I have <b>some</b></>
</>
```

Note that `div`s and `span`s are by definition already semantic-less. And any semantic HTML element that expects certain children elements will strip their semantics as well.


### Globally inherited states and properties

Every element inherits some [global states and properties](http://www.w3.org/TR/wai-aria/states_and_properties#global_states) that describe it’s... well, states and properties.

Looking back at our accessibility tree, we still need to address the hierarchal issue. The elements sit as two disparate DOM elements with no connection to each other. However, we want to describe a parent/child relationship between the `input` and the `table` to appropriately represent the composite widget.

So, we use the [`aria-owns` property](http://www.w3.org/TR/wai-aria/states_and_properties#aria-owns) by passing a space separated list of element IDs that it “contains” but cannot already be semantically inferred from the DOM:

```html
<input aria-owns="datepicker_table">
<table role="grid" id="datepicker_table">
	<tr role="row">
		...
		<td role="gridcell">19</td>
		<td role="gridcell">20</td>
		...
	</tr>
</table>
```

When the screen reader now comes across this markup, it renders the following accessibility tree:

```html
<input>
	<grid>
		<row>
			...
			<gridcell></gridcell>
			<gridcell></gridcell>
			...
		</row>
	</grid>
</input>
```

That’s more like it. Now the `grid` is “contained” by the `input` as we intended it to.


### Role inherited states and properties

So far we’ve only talked about attributes that allow us to define semantics and relationships between types of elements. But UI components also inherit further states and properties based on their `role` definition. These states must be updated as the user interacts with the widget.

To fix our date picker example, we need to make two additions that will describe:

1. That the `gridcell`s “control” the value of the `input element, and
2. Which `gridcell` is currently selected.

To do so, the `gridcell` role inherits the [`aria-controls` property](http://www.w3.org/TR/wai-aria/states_and_properties#aria-controls) (that expects the ID of an element whose value it controls) and the [`aria-selected` state](http://www.w3.org/TR/wai-aria/states_and_properties#aria-selected) (that determines it’s selected state):

```html
<input id="datepicker" aria-owns="datepicker_table">
<table role="grid" id="datepicker_table">
	<tr role="row">
		...
		<td role="gridcell" aria-controls="datepicker">19</td>
		<td role="gridcell" aria-controls="datepicker" aria-selected="true">20</td>
		...
	</tr>
</table>
```

This tells the screen reader that the UI component the `gridcell` controls has this option as “selected”.

We’re now starting to scratch the surface of what’s capable in building an interactive accessible widget. There’s a whole list of more [widget](http://www.w3.org/TR/wai-aria/states_and_properties#attrs_widgets) and [global](http://www.w3.org/TR/wai-aria/states_and_properties#global_states) states and properties that we can use to make this a fuller and more complex widget.

For example, if the picker actually expands/collapses on focus/blur, then we would represent that interaction using [`aria-haspopup`](http://www.w3.org/TR/wai-aria/states_and_properties#aria-haspopup) and [`aria-hidden`](http://www.w3.org/TR/wai-aria/states_and_properties#aria-hidden) as such:

```html
<input id="datepicker" aria-owns="datepicker_table" aria-haspopup="true">
<table id="datepicker_table" role="grid" aria-hidden="true">
	<tr role="row">
		...
		<td role="gridcell" aria-controls="datepicker">19</td>
		<td role="gridcell" aria-controls="datepicker" aria-selected="true">20</td>
		...
	</tr>
</table>
```

And then switch the `table`’s `aria-hidden` to `false` when the picker opens.


## Learn by doing

This is a very basic example with a lot of room to add functionality and improve based on the interaction desired. I would suggest going through the list of [all role definitions](http://www.w3.org/TR/wai-aria/roles#role_definitions) and the list of [all states and properties](http://www.w3.org/TR/wai-aria/states_and_properties#state_prop_def) that can be used to define semantics.

As authors of tools and technologies that are built to help the community, it is our responsibility to make sure we produce solutions that are accessible to everyone.

The current state of the web is scarily inaccessible because of the false notion that implementing accessibility features is complicated and difficult. Well yeah, it *does* requires some extra effort, but the benefits reaped far outweigh the effort needed. So let’s do our part in building a more inclusive community on the web.


---

Links:

- [ARIA roles categorization](http://www.w3.org/TR/wai-aria/roles#roles_categorization)
- [ARIA states’ and properties’ taxonomy](http://www.w3.org/TR/wai-aria/states_and_properties#state_prop_taxonomy)
- [ARIA Design Patterns](http://www.w3.org/WAI/PF/aria-practices/#aria_ex)
- [My notes on ARIA](http://amsul.ca/notepad/aria/)
- [Nu Markup Checker](http://validator.w3.org/nu/) - online accessibility validator
- [NVDA](http://www.nvaccess.org/) - open source screen reader

Tags: ARIA, Accessibility

