# 前言

这套方案的核心观点是不要把git的commit顺序当成版本的顺序，而是进一步的细分，将commit中的每一个文件的修改部分当成是一种带有版本先后顺序。git的commit不再是顺序的向下延伸不断前进，而是作为一个个独立的节点，就像是海滩上一个个的贝壳，是零碎的存在，彼此间联系甚微，将某个文件放到另外一个版本上的行为，更加像是在海滩上挑选贝壳，将他们扔到一个桶中，而这个桶就是下一个环境。

在这种想法的催生下，可以衍生出一种畸形的git使用方式，这种方式可以适应多个环境下有多个互不相关的版本的问题。用人类阅读的语言就是说可以同时开发3-4个特性同时开发，却在完全先后顺序的条件下登录不同的环境，当然，其中的版本关联问题也是需要处理的。

想法基于主干分支开发，对CI（持续集成）的过程友好，并且在大型项目中，也建议尽早进行合并。

当然，因为使用极端特殊，这个文档是和分享作为将过程脚本化，自动化的依据存在的，使用这个方案管理的难度以及复杂度和其他的git方案相比是量级差距的提升，甚至需要使用数据库维护信息。这个文档更加接近预想手册。

# 仅仅2个环境的情况

下面我们将情况简化到只有2个环境，为了方便一个环境成为SIT（系统集成）另一个成为PRD（生产环境），之后我们会从这里面推到出3个乃至更多环境应该如何进行调度和使用。

## 完全无关联的情况，3个特性同时开发（如何开发）

这里我们的假设是3个特性将同时进行开发，但是他们并没有编辑到同一个文件，也就没有任何合并冲突问题出现。

这是最简单的情况，主要演示的是如何在这种思想下进行从开发者本地集成到公共环境、如何将环境安装到另外一个版本上。这一节探讨的问题很弱智，但是麻烦看一看，因为是后面一些情况推演的基础。

### 现在的环境

如先前所述，我们假设有两个环境，sit和prd环境，这两个环境是主线环境，我们需要针对这两个环境进行基于特性的版本控制。

![image-20200908003355308](.\picture\image-20200908003118079.png)

我们假设当前的环境如<u>*上*</u>*<u>图</u>*所示，**ft01**代表了我们开发的东西属于**特性01**，后面的是表明这个合并到主分支上面的commit信息。~~原谅我很没有创意的用古诗词作为讲解道具，因为这东西没有什么阅读障碍，都是小学课本上的东西，后文可能会勾起你高中的不好回忆。~~

目前的状态开发如下图，登鹳雀楼完成了，江雪完成了第一段。

![image-20200908003948035](.\picture\image-20200908003751286.png)

### 特性如何开发，从DEV到SIT

下面我们将用开发者视角，开发一个新的特性——塞下曲，属于feature03.

下面是简易的开发步骤

第一步，在环境最新的版本上开启一个分支。

```
git checkout -b ft03
```

第二步，在你的特性分支上任意开发，commit的内容不作出过度的要求，如果乐意，将分支上传也是可以的。

```
git add .
git commit -m '你想提交什么请随意，在你的特性分支上受限制'
git push
```

你的作品完成了第一句话` 月黑雁飞高，单于夜遁逃。` 下图是你在**你的分支**上commit完成的结果

![image-20200908005238906](.\picture\image-20200908005238906.png)

![image-20200908004918637](.\picture\image-20200908004918637.png)

这个时候你准备将你的成果添加到sit环境中，不过还记得江雪吗？它被别人完成了，并且先你一步提交了服务器，不过不要紧，你们在不同的文件上，并不会产生版本冲突。这个时候的环境开启来是这样的。注意，我们约定进入环境主要分支的commit活动的备注内容需要用ft+数字开头，这样方便日后的管理工作，至于如果不遵守，建议招收版本管理员的时候选择——人高马大体大腰圆孔武有力说一不二的（进行物理和生理说服）。

![image-20200908005542841](.\picture\image-20200908005542841.png)

#### 如何合并回主线，这是重头戏！！！

你想要的将你的代码合并到主干上。

所以，首先用merge确保一下你和环境间的冲突吧，这一步也是处理你和大环境之间的版本关联的。

```
git checkout ft03 # 保证自己在特性的分支上
# git pull
git merge sit # 将主干分支的更新内容加入自己的分支
```

完成后情况如下图

![image-20201027001154626](F:\git研发\multienv-multiver-gittool\theory\picture\image-20201027001154626.png)

![image-20200908010256999](.\picture\image-20200908010256999.png

![image-20201027001110249](F:\git研发\multienv-multiver-gittool\theory\picture\image-20201027001110249.png)



其实并没有版本关联（肯定的），不过这个时候江雪已经到了你的分支上了，然后尽快将你的分支merge进入sit环境，这样才算你的代码进入主线，不过注意，合并回主线的merge命令需要特别对待了

```
git checkout sit
git merge --squash ft03
git add .
git commit -m 'ft~~-你提交的内容'
```

记住，使用merge的squash 模式，区别可以看下图，相当于是直接进ft03分支上的全部变更直接复制到了sit分支上，很像rebase命令的效果，但是保留了commit的信息，不是那种找打的在远程分支上rebase的三流程序员。

![image-20201027002208854](C:\Users\85341\AppData\Roaming\Typora\typora-user-images\image-20201027002208854.png)

同时注意，commit的名字也要符合之前说的要求**ft数字-commit的内容**

如果忽视ft03分支，sit分支上可以看成这样

![image-20201027004758155](F:\git研发\multienv-multiver-gittool\theory\picture\image-20201027004758155.png)



#### 后日谈

如果要持续集成，尽量避免使用分支。然而因为这种畸形的使用merge，可以做到尽早合并，尽快合并。意见每天下班前，如果自己的dev环境可靠，就进行一次合并到主干的操作，保证提后不要出现太多的版本关联问题。

当然在这种使用模型下面，一个future并不是合并完就死掉了（和rebase有明显区别），它还可以继续进行接下来的开发，然后将主干融入再融合进入主干，比如这样。而且我们建议，在下图中合并的时候，生成一个空的commit，标志自己进入主干分支 ` git commit --allow-empty -m "合并到主干分支"`

![image-20201027004643344](F:\git研发\multienv-multiver-gittool\theory\picture\image-20201027004643344.png)

不过这种merge squash开发有着一个问题：你的修改是无源头的，这个对于git来说一个很不爽的事情——你的一切修改，使用了squash合并之后，原则上讲都和之前的commit行为脱钩了。最简单的讲，git认为sit上的提交和ft03上的提交不是同一个东西，如果日后修改了这个文件git会认为是个冲突而不是来源你的修改，所以还要讲自己的提交merge回自己在的特性分支上。

```shell
git checkout ft03
git merge sit
```

![image-20201028010643647](F:\git研发\multienv-multiver-gittool\theory\picture\image-20201028010643647.png)

这种多段多次的各种方式的merge是我们的交流基础和共识，如果你认同了这个观点，下面的故事才可能发生，不然就会是无穷地争论方便性和管理了，接下来的所有commit，我们都认为可能是有着大量的复杂文件提交且合并了一个最终版本才提交的。

这个git模型连作者本人都觉得麻烦，但是它也适合的是极度麻烦的环境——拥有多个环境，每个环境还有各自的版本，每个环境间需要的特性甚至前后顺序都是颠倒错位的，甚至有的特性上了某个环境运行一阵之后又要撤下来。

极端的环境造就极端的生物，这种git的管理甚至需要使用到数据库以及脚本才能勉强运行，但是它确实适合应对这种极端的环境要求，满足复杂环境下对特性的更新。



### 从一个环境到另一个环境，也是建议

进行从一个环境到另一个环境的迁移，这里使用的是cherry-pick命令，这也是之前我们用贝壳比喻的原因——就是从某个环境中挑选出一粒，然后pick到下一个环境上。

不过，在cherry-pick前，我们还做一件事——确定这次cherrypick带来的版本关联。也就是如果想要将某个commit亦或是某个feature下的全部commit移动到另一个环境，需要去进行这一系列commit中的文件和其他commit之间的版本关联问题，如果不幸产生了版本关联，那么只能进行人工处理了。

目前的状况如下图所示

![image-20201028011517608](F:\git研发\multienv-multiver-gittool\theory\picture\image-20201028011517608.png)



现在我们假设ft01需要登录prd环境，首先确定目前存在有哪些commit属于ft01，你可以输入这行

`  git log --pretty=oneline --grep "ft01"`

![image-20201028012802103](F:\git研发\multienv-multiver-gittool\theory\picture\image-20201028012802103.png)

> 参考资料：https://git-scm.com/book/zh/v2/Git-%E5%9F%BA%E7%A1%80-%E6%9F%A5%E7%9C%8B%E6%8F%90%E4%BA%A4%E5%8E%86%E5%8F%B2

现在我们有了所有的ft01的commit信息（只有一条）

接下来就是判断每个commit中提交的全部文件了。

`git diff-tree --no-commit-id --name-only -r <commit-hash>`

![image-20201028013347025](C:\Users\85341\AppData\Roaming\Typora\typora-user-images\image-20201028013347025.png)

在下一步，就是判断commit是否产生了关联，最新版本的文件是不是将其他版本的特性带了上去

`git log --pretty=oneline 文件名`

![image-20201028013702580](C:\Users\85341\AppData\Roaming\Typora\typora-user-images\image-20201028013702580.png)

发现没有关联（commit 之前没有出现不用上新环境的文件）之后就使用cherry-pick将所有的文件移动到下一个环境中，也就搞定了文件的读取了。

```
git checkout prd
git cherry-pick <commit-hash>
```

下面演示将ft02进入prd环境

```
git log --pretty=oneline --grep "ft01"

git diff-tree --no-commit-id --name-only -r 31f9e58adcd9a1b48af2f2d31d2b4a63e654918d
git diff-tree --no-commit-id --name-only -r 70785cf9cc67fe268fe64f6f5783f6716efd1b7f

git log --pretty=oneline 江雪.txt
git log --pretty=oneline 鹳雀楼.txt

git cherry-pick 70785cf9cc67fe268fe64f6f5783f6716efd1b7f
git cherry-pick 31f9e58adcd9a1b48af2f2d31d2b4a63e654918d
```



![image-20201028015149248](C:\Users\85341\AppData\Roaming\Typora\typora-user-images\image-20201028015149248.png)

![image-20201028015019819](F:\git研发\multienv-multiver-gittool\theory\picture\image-20201028015019819.png)



## 发生冲突的情况





## 发生冲突特性移动环境前后颠倒的情况

