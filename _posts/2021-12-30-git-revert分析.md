---
title: "git : revert分析"
categories:
  - git 
tags:
  - git
  - revert
  - conflict
---

## 问题描述：修改分支合并目标，为什么需要两次revert

<!-- git-revert 图片，来自draw.io -->
<div class="mxgraph" style="max-width:100%;border:1px solid transparent;" data-mxgraph="{&quot;highlight&quot;:&quot;#0000ff&quot;,&quot;nav&quot;:true,&quot;resize&quot;:true,&quot;toolbar&quot;:&quot;zoom layers tags lightbox&quot;,&quot;edit&quot;:&quot;_blank&quot;,&quot;xml&quot;:&quot;&lt;mxfile host=\&quot;app.diagrams.net\&quot; modified=\&quot;2021-12-30T10:52:08.676Z\&quot; agent=\&quot;5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.110 Safari/537.36\&quot; etag=\&quot;fc3pcRRV_T_MYh57SW15\&quot; version=\&quot;16.1.0\&quot; type=\&quot;device\&quot;&gt;&lt;diagram id=\&quot;nqz5p0M37df7dpH7cNmB\&quot; name=\&quot;Page-1\&quot;&gt;7Zpdc6IwFIZ/DdOrdoCAymWrtnvRndlpZ3bt1U4qEbILhAlRsb9+E0n4EEV315Z02hslh3xx3pPHk6ABxnF+R2EafiU+igzb9HMDTAzb9pwB/xSGTWFwbWkIKPYLk1UZHvELkkZTWpfYR1mjIiMkYjhtGuckSdCcNWyQUrJuVluQqDlqCgPUMjzOYdS2/sA+CwvryB5W9i8IB6Ea2Rp4xZ0YqsrySbIQ+mRdM4GpAcaUEFZcxfkYRcJ3yi9Fu9sDd8uJUZSwUxoQz/GmqzyNEzZ4mD1sxpfff18CNTm2UU+MfO4AWUxIwr9uKFkmPhL9mLxEKAtJQBIY3ROScqPFjb8QYxspH1wywk0hiyN5F+WYzUTzK1eWnmRn4nqS1wsbVUgY3cxUB6LwVPUgilWzbUm182EWbucq2rWdJP2WkSWdow7PqGCDNECso56UW3itNoCU4A6RGPHJ8QoURZDhVTOsoIzOoKxXCcgvpIZ/o6fVo57WaXrWJHxqKKiFnq5Wesp+VzBaypGmbYGjiNNQ6LoOMUOPKdw6Ys153FQNZmmByAXOhUcPe3OFKEN55/PLu85AEkTyvQT3uqKlK01hDZTKdnaPifZvuQLqPCtXw3mJ9u9Bb58Y9COtgt5uBf21XkFvm82gB2bPQW95nUFfxfe0sn6UNQDeJfhVmlxbBBSJCN2Pt3v4zPPuZuBHOEj49Zx7DFFuEK0xz2yv5Y0Y+34RCCjDL/B525/wdUpwwrYP5N4Y7mSv9zvDsLWSyvxcjtJIgfetMPPKAkAqd7LDZW/fxPRrVchikXHldxUpB/2PbKsNqosLvVA1UL+H2qBq+ImqIwg6iqqhXqhyDqBq/15EE1QNz4iqkTdsLDKVtutLLncPufQC1+7Gon9wuZ/gOgKko+Cy9QIX6FS0z8Ov15dC7dM10ULhuALSWC8e2Z6jF49GLY/N9PKYtbNLLk/CeyO480nwQ77x3ic27DY3npfBTx7Fe6XWJPdUCfMZck9zALzmKjM0Tz29lmITvbgFhk1uWU7fR9pv/FJHa1ApAL2zM2017VrY3+gV9ruH2r2HPdA3Pe/53bQ9OnENaPZyujv/+sAvp08WFGglqN3eg9zqBbXd4+9XfD3Ni9WfeYpkqfpHFJj+AQ==&lt;/diagram&gt;&lt;/mxfile&gt;&quot;}"></div>
<script type="text/javascript" src="https://viewer.diagrams.net/js/viewer-static.min.js"></script>

### 说明：
* x是原始节点，上面是功能分支，下面是目标分支

### 背景：
* A/B 首次合并产生C，发现C有问题，执行revert1 生成A'然后在原来的功能分支上执行bug修复，产生D节点

### 现象及分析：
1. 如果这个时候合并D和A'，生成E节点。那么这次执行三相合并的时候，公共节点是B。那么D相对于B，是修改了一部分东西，A'相对于B，是删除了原来B到C的全部内容。所以他们合并的时候，原来B修改的东西，就会被一律认为是删除操作，或者D改了原来B的，那么就会冲突（D想修改，A'想删除）。
2. 这个时候执行一次revert2操作，生成A'的节点，这个时候其实是把A'删掉的内容还原回来。这个时候A'此时就含有了B的内容。执行A"'和D的合并，生成F节点。这个时候，公共节点仍然是B。合并口和A"的时候，A'跟B相比，其实没有差别，就是没动过了，那么修改点就只剩D的修改内容，所以B里的东西就不会丢。

### 结论：
* 上面这个简化场景情況比较简单，指的是B改错了一点东西不小心合并进了目标分支，目标分支这个时候可能有其他人在用为了防止影响别人，先revert掉。开发者自己修复问题，就必须再revert 一次，才能把自己的修改准确的带进去。
* 其实，假如这个时候目标分支没人用，你发现合并错误了，其实是可以在自己分支上改，然后再合并一次，不需要引入这个r
evert， 但是目标分支没人用这个前提，目标分支如果是uat dev这些，就很麻烦了。因此，建议一旦发现合并出了问题，通常这个问题应该很快就能发现，最好就直接把目标分支reset --hard 到× 节点（如果之后有其他人的合并，一般不会太多，麻烦一点就再重新提merge reques就行），自己分支上改完了，重新往X上去合并，不要做n次revert导致在分支里引入大量记录，后面乱
* 另外再补充下，不要用gitlab的线上冲突解决，比如你的feature和uat冲突，可能就一行有问题，web端选use theirs选项，服务
端实质上是把uat直接往你的feature里合并-- 下，直接就污染了你的feature，以后你的feature往别的分支合并，问接的就带上