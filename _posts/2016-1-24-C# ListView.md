---
layout: post
author: tnqiang
titile: Unity ListView
category: Unity
tag: Unity
---
##About
This is a ListView solution for Unity’s new UI(uGUI) system.
This solution provides following conveniences:

- Attach scrollview related components automatically
- Resize scrollview length as desired automatically
- Reuse list items gameobject
- Easy to use and easy to extend

##Usage

1. Create a list item prefab;
2. Create a Class derived from IUListItemView and implement its SetData function;
3. Attach the Class above to list item prefab;
4. Create an empty ui gameobject (with RectTransform attached) to show the listview;
5. Attach an IUListView implementation, eg. USimpleListView;
6. Set propertities of the IUListView subclass;
7. Create another MonoBehavior Class to invoke the IUListView's SetData function;
8. Run.

##Plan

1. USimpleListView: provides simple horizontal/vertical layout, one item per row/column (Completed);
2. UGridListView: provides grid horizontal/vertical layout, allowing multi items per row/column (Completed);
3. USimpleDiffSizeListView: provides simple horizontal/vertical layout, one item per row/column, each item has a different size (Completed).

##Author
tnqiang

##License
MIT

##Link:
[Unity_ListView](https://github.com/tnqiang/Unity_ListView)

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<!-- TitleAds -->
<ins class="adsbygoogle"
     style="display:inline-block;width:728px;height:90px"
     data-ad-client="ca-pub-4576025583130785"
     data-ad-slot="1109364752"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>