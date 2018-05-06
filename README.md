# Markdown Sandbox
This is an area to experiment with GitHub markdown.
 
 
<details>
<summary>"Table of Contents"</summary>
Anchors<br>
Text<br>
Headers<br>
Carriage Returns<br>
Pre-formatted Text<br>
Spacing<br>
Bullets<br>
Tables<br>
Paragraphs<br>
Source Code<br>
Images<br>
Resources<br> 
</details>


# Anchors
[Jump to anchor named 'text'](#text)<br>
[Jump to anchor named 'headers'](#headers)<br>
[Jump to anchor named 'carriage_returns'](#carriage_returns)<br>
and so on...

```
[Jump to anchor named 'foo'](#foo)
...
# <a name="foo"></a>Header goes here

```

# <a name="text"></a>Text
This is plain text.\
This is *italisized*.\
This is **bolded**\
This is ***bolded and italisized***

```
This is plain text.\
This is *italisized*.\
This is **bolded**\
This is ***bolded and italisized***
```


# <a name="headers"></a>     Header 1
##     Header 2
###    Header 3
####   Header 4
#####  Header 5
###### Header 6

```
#      Header 1
##     Header 2
###    Header 3
####   Header 4
#####  Header 5
###### Header 6
```

# <a name="carriage_returns"></a>Carriage Returns
This is a really long sentence that just got cut off\
OUCH!

```
This is a really long sentence that just got cut off\
OUCH!
```

# <a name="pre-formatted_text"></a>Pre-formatted Text
<pre>
This is pre-formatted text.
No breaks, paragraphs, or backslashes are used to produce a carriage return.
What you see is what you get.
</pre>

# <a name="spacing"></a>Spacing
The beginning of this sentence has no spaces followed by a backslash \(\\\).\
 The beginning of this sentence has 1 space followed by a backslash \(\\\).\
  The beginning of this sentence has 2 spaces followed by a backslash \(\\\).\
   The beginning of this sentence has 3 spaces followed by a backslash \(\\\).\
    The beginning of this sentence has 4 spaces with NO backslash at the end.

Spacing -- doesn't do jack. Everything gets aligned to the left.

```
The beginning of this sentence has no spaces followed by a backslash \(\\\).\
 The beginning of this sentence has 1 space followed by a backslash \(\\\).\
  The beginning of this sentence has 2 spaces followed by a backslash \(\\\).\
   The beginning of this sentence has 3 spaces followed by a backslash \(\\\).\
    The beginning of this sentence has 4 spaces with NO backslash at the end.
```

# <a name="bullets"></a>Bullets
* Bullet 1
* Bullet 2
  * Indent 1
    * Indent 2

```
* Bullet 1
* Bullet 2
  * Indent 1
    * Indent 2
```


# <a name="hyperlinks"></a>Hyperlinks 
[Example_1]: https://cloud.google.com \
[Example_2]: https://cloud.google.com/compute/docs/vpn/overview \
[Example_3](https://cloud.google.com/sdk/gcloud/reference/compute/routers/update-bgp-peer) \
Go to the [Example 4](https://console.cloud.google.com/networking/firewalls) page 

```
[Example_1]: https://cloud.google.com \
[Example_2]: https://cloud.google.com/compute/docs/vpn/overview \
[Example_3](https://cloud.google.com/sdk/gcloud/reference/compute/routers/update-bgp-peer) \
Go to the [Example 4](https://console.cloud.google.com/networking/firewalls) page 
```

# <a name="tables"></a>Tables

| key | value |
------|--------
| apple | green |
| cherry | red |


```
| key | value |
------|--------
| apple | green |
| cherry | red |
```

# <a name="paragraphs"></a>Paragraphs
1. First Paragraph \ 
   This is the first sentence.\ 
   This is the second sentence. \

1. Second Paragraph \
   This is the third sentence. \
   This is the fourth sentence. \

```
1. First Paragraph \ 
   This is the first sentence.\ 
   This is the second sentence. \

1. Second Paragraph \
   This is the third sentence. \
   This is the fourth sentence. \
```

# <a name="source_code"></a>Source Code

```
$ yum install update
```

```python
print 'hello, world!'
# Comment
```


# <a name="images"></a>Images
![](./images/button.png)
<img align="left" src="./images/button.png" height="200" />Image aligned to the left
<hr size = "10"> 
<img align="right" src="./images/button.png" height="200" />Image aligned to the right
<hr size = "10">
<img align="center" src="./images/button.png" height="200" />Image aligned to the center


```
![](./images/button.png)
<img align="left" src="./images/button.png" height="200" />Image aligned to the left
<img align="right" src="./images/button.png" height="200" />Image aligned to the right<br>
<img align="center" src="./images/button.png" height="200" />Image aligned to the center
```

# <a name="resources"></a>Resources
