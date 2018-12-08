>头大！挑战杯项目用到一个item-user的算法，就先简单的看看UserCF吧，原理上还是不难，实现上有点难度。针对计算用户间的相似度，类似与KNN或者Kmeans。😗

## 基于User的协同过滤算法

>UserCF：User  Collaboration   Filter，基于用户的协同过滤.

#### 核心思想：

在一个在线推荐系统中，当用户A需要个性化推荐时，系统先找到和他有相似兴趣的其它用户，然后把这些用户喜欢的、而用户A没有听说过的物品推荐给A，这种方法称为基于用户的协同过滤算法。

![](https://i.imgur.com/lElnYGg.png)

图中，user1购买了 1 2 3 4 user2购买了2 user3购买了2 3 ，相比于user2他更合user1相似，于是将user1购买的而user3没有购买的东西推荐给user3.

得到如下稀疏矩阵User-item：
| item |p1 |p2| p3 | p4 |
|--|--|--|--|--|
| user1 |1  |1 | 1 | 1 |
| user2 | 0 | 1|  0|  0|
| user3 | 0 |1 |  1| 0 |

为什么说他是稀疏矩阵？比如你在淘宝购物，你这么些年就买了几十样物品，而淘宝提供的物品成千上万，因此你的很多item对应的是0.由此称为稀疏举证。

* 从中可以看出，这个算法主要包括两步：

	* **找到和目标用户兴趣相似的用户集合——计算两个用户的兴趣相似度。**

	* **找到这个集合中的用户喜欢的，且目标用户没有听说过的物品推荐给目标用户。**
	
在基于User的CF中需要吧UI矩阵变为UU矩阵。=》UU=UI*IU  计算后一般需要归一化[0,1] 。下图抽取跟C更加相关的人物BD来做评分预测。最后得到4.05分表示C对Titanic还算比较感兴趣，于是可以进行推送。这里需要提到的一点是，这里直接找到了最匹配的两个User 来预测评分，和我最后的代码有一点点区别。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181206135011661.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTE1ODMzMTY=,size_16,color_FFFFFF,t_70)

#### 具体实现

**计算用户间相似度**

>CF的常用方法有三种，分别是欧式距离法、皮尔逊相关系数法、余弦相似度法。

1. 皮尔逊相关系数
皮尔逊相关系统是比欧几里德距离更加复杂的可以判断人们兴趣相似度的一种方法。它在数据不是很规范时，会倾向于给出更好的结果。
假定有两个变量X、Y，那么两变量间的皮尔逊相关系数可通过以下公式计算：
![](https://i.imgur.com/yo3zvX8.png)

其中E是数学期望，cov表示协方差，N表示变量取值的个数。
2. 欧几里德距离
假定两个用户X、Y，均为n维向量，表示用户对n个商品的评分，那么X与Y的欧几里德距离就是：


![](https://i.imgur.com/qsRi45d.png)



数值越小则代表相似度越高，但是对于不同的n，计算出来的距离不便于控制，所以需要进行如下转换：

![](https://i.imgur.com/8MUhe2C.png)


使得结果分布在(0,1]上，数值越大，相似度越高。

3. 余弦距离


余弦距离，也称为余弦相似度，是用向量空间中两个向量余弦值作为衡量两个个体间差异大小的度量值。
与前面的欧几里德距离相似，用户X、Y为两个n维向量，套用余弦公式，其余弦距离表示为：

![](https://i.imgur.com/sAz4A7j.png)


即两个向量夹角的余弦值。但是相比欧式距离，余弦距离更加注意两个向量在方向上的相对差异，而不是在空间上的绝对距离，具体可以借助下图来感受两者间的区别：

![](https://i.imgur.com/OOjKagp.png)



余弦距离与欧式距离的区别如上图。

**推荐物品**

在选取上述方法中的一种得到各个用户之间相似度后，针对目标用户u，我们选出最相似的k个用户，用集合S(u,k)表示，将S中所有用户喜欢的物品提取出来并去除目标用户u已经喜欢的物品。然后对余下的物品进行评分与相似度加权，得到的结果进行排序。最后由排序结果对目标用户U进行推荐。其中，对于每个可能推荐的物品i，用户u对其的感兴趣的程度可以用如下公式计算：


![](https://i.imgur.com/tqFDSfm.png)



其中，p(u,i)表示用户u对物品i的感兴趣程度，S(u,k)表示和用户u兴趣最接近的K个用户，N(i)表示对物品i有过行为的用户集合，Wuv表示用户u和用户v的兴趣相似度，Rvi表示用户v对物品i的兴趣.

## 缺陷

虽然协同过滤是一种比较省事的推荐方法，但在某些场合下并不如利用元信息推荐好用。协同过滤会遇到的两个常见问题是

1)稀疏性问题——因用户做出评价过少，导致算出的相关系数不准确

2)冷启动问题——因物品获得评价过少，导致无“权”进入推荐列表中

因此在对于新用户和新物品进行推荐时，使用一些更一般性的方法效果可能会更好。比如：

* 给新用户推荐更多平均得分超高的电影；
* 把新电影推荐给喜欢类似电影（如具有相同导演或演员）的人。


后面这种做法需要维护一个物品分类表，这个表既可以是基于物品元信息划分的，也可是通过聚类得到的。

**附录：**

长尾分布 

>长尾效应，英文名称Long Tail Effect。从人们需求的角度来看，大多数的需求会集中在头部，而这部分我们可以称之为流行，而分布在尾部的需求是个性化的，零散的小量的需求。而这部分差异化的、少量的需求会在需求曲线上面形成一条长长的“尾巴”，而所谓长尾效应就在于它的数量上，将所有非流行的市场累加起来就会形成一个比流行市场还大的市场。


![](https://i.imgur.com/w55kWcb.png)

如：用户越活跃越倾向于浏览冷门的物品

![](https://i.imgur.com/jCAdITW.png)


## Java实现
>代码是借鉴别人的，他这里面是计算了P5与其他所有用户的相关系数，通过相关系数矩阵与User-item矩阵相乘计算推荐度。最后进行一个Top N的取值，取相推荐度最高的一个item进行推荐。
```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.Comparator;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Map.Entry;

//JAVA基于用户的协同过滤实现
/**
 * UserCF的核心思想即为根据用户数据模拟向量相似度，我们根据这个相似度，
 * 来找出指定用户的相似用户，然后将相似用户买过的而指定用户没有买的东西推荐给指定用户，
 * 推荐度的计算也是结合了相似用户与指定用户的相似度累加。注意这里我们默认是用户的隐反馈行为
 * 
 * @author 丽丽超可爱
 *
 */



public class testRecommend {

	public static void main(String[] args) {
		Map<String, Map<String, Integer>> userPerfMap = new HashMap<String, Map<String, Integer>>();
		Map<String, Integer> pref1 = new HashMap<String, Integer>();
		pref1.put("A", 3);
		pref1.put("B", 4);
		pref1.put("C", 3);
		pref1.put("D", 5);
		pref1.put("E", 1);
		pref1.put("F", 4);
		userPerfMap.put("p1", pref1);
		Map<String, Integer> pref2 = new HashMap<String, Integer>();
		pref2.put("A", 2);
		pref2.put("B", 4);
		pref2.put("C", 4);
		pref2.put("D", 5);
		pref2.put("E", 3);
		pref2.put("F", 2);
		userPerfMap.put("p2", pref2);
		Map<String, Integer> pref3 = new HashMap<String, Integer>();
		pref3.put("A", 3);
		pref3.put("B", 5);
		pref3.put("C", 4);
		pref3.put("D", 5);
		pref3.put("E", 2);
		pref3.put("F", 1);
		userPerfMap.put("p3", pref3);
		Map<String, Integer> pref4 = new HashMap<String, Integer>();
		pref4.put("A", 2);
		pref4.put("B", 2);
		pref4.put("C", 3);
		pref4.put("D", 4);
		pref4.put("E", 3);
		pref4.put("F", 2);
		userPerfMap.put("p4", pref4);
		Map<String, Integer> pref5 = new HashMap<String, Integer>();
		pref5.put("A", 4);
		pref5.put("B", 4);
		pref5.put("C", 4);
		pref5.put("D", 5);
		pref5.put("E", 1);
		pref5.put("F", 0);
		userPerfMap.put("p5", pref5);
		
		/*	A	B	C	D	E	F
		P1	3	4	3	5	1	4
		P2	2	4	4	5	3	2
		P3	3	5	4	5	2	1
		P4	2	2	3	4	3	2
		P5	4	4	4	5	1	0*/
		
		Map<String, Double> simUserSimMap = new HashMap<String, Double>();

		String output1 = "皮尔逊相关系数:", output2 = "欧几里得距离:";
		// 遍历userMap里面的数据
		for (Entry<String, Map<String, Integer>> userPerfEn : userPerfMap.entrySet()) {
			String userName = userPerfEn.getKey();
			if (!"p5".equals(userName)) {
				// 计算p5用户与其他用户的相关系数，越大越好  得到 如 A  4 的key-value对 
				double sim = getUserSimilar(pref5, userPerfEn.getValue());
				// 计算p5用户和其他用户的欧氏距离
				double distance = getEuclidDistance(pref5, userPerfEn.getValue());
				output1 += "p5与" + userName + "之间的相关系数:" + sim + ",";

				output2 += "p5与" + userName + "之间的距离:" + distance + ",";
				// 保存p5与其他每个用户的相关系数
				simUserSimMap.put(userName, sim);
				// System.out.println(simUserSimMap.get(userName));
			}

		}
		
		/*	P1		P2		P3		P4		
		P5	0.43	0.66	0.918	0.36*/
		// 选出k个相关系数较高的user

		// 输出相关系数与结果
		System.out.println(output1);
		System.out.println(output2);
		// 新创建一组新用户偏好电影 评分数据  map
		Map<String, Map<String, Integer>> simUserObjMap = new HashMap<String, Map<String, Integer>>();
		Map<String, Integer> pobjMap1 = new HashMap<String, Integer>();
		pobjMap1.put("一夜惊喜", 3);
		pobjMap1.put("环太平洋", 4);
		pobjMap1.put("变形金刚", 3);
		simUserObjMap.put("p1", pobjMap1);
		Map<String, Integer> pobjMap2 = new HashMap<String, Integer>();
		pobjMap2.put("一夜惊喜", 5);
		pobjMap2.put("环太平洋", 1);
		pobjMap2.put("变形金刚", 2);
		simUserObjMap.put("p2", pobjMap2);
		Map<String, Integer> pobjMap3 = new HashMap<String, Integer>();
		pobjMap3.put("一夜惊喜", 2);
		pobjMap3.put("环太平洋", 5);
		pobjMap3.put("变形金刚", 5);
		simUserObjMap.put("p3", pobjMap3);
		/*		一夜惊喜		环太平洋		变形金刚
		 * P1	3			4			3	
		 * P2	5			1			2
		 * P3	2			5			5
		 * */
		// 两个矩阵进行预测计算
		/*	P1		P2		P3		P4		
		P5	0.43	0.66	0.918	0.36*/
		// 参数一  三个用户的电影喜好评分       参数二 P5的与其他用户的相关系数
		System.out.println("根据系数推荐:" + getRecommend(simUserObjMap, simUserSimMap));
	}

	/**
	 * 
	 * @Title getUserSimilar
	 * @Class testRecommend
	 * @return double
	 * @param pm1
	 * @param pm2
	 * @return
	 * @Description获取两个用户之间的皮尔逊相似度,相关系数的绝对值越大,相关度越大
	 * 
	 * @Date 2013-9-4
	 */

	public static double getUserSimilar(Map<String, Integer> pm1, Map<String, Integer> pm2) {
		int n = 0;// 数量n
		int sxy = 0;// Σxy=x1*y1+x2*y2+....xn*yn
		int sx = 0;// Σx=x1+x2+....xn
		int sy = 0;// Σy=y1+y2+...yn
		int sx2 = 0;// Σx2=(x1)2+(x2)2+....(xn)2
		int sy2 = 0;// Σy2=(y1)2+(y2)2+....(yn)2
		for (Entry<String, Integer> pme : pm1.entrySet()) {
			String key = pme.getKey();
			Integer x = pme.getValue();
			Integer y = pm2.get(key);
			if (x != null && y != null) {
				n++;
				sxy += x * y;
				sx += x;
				sy += y;
				sx2 += Math.pow(x, 2);
				sy2 += Math.pow(y, 2);
			}
		}
		// p=(Σxy-Σx*Σy/n)/Math.sqrt((Σx2-(Σx)2/n)(Σy2-(Σy)2/n));
		double sd = sxy - sx * sy / n;
		double sm = Math.sqrt((sx2 - Math.pow(sx, 2) / n) * (sy2 - Math.pow(sy, 2) / n));
		return Math.abs(sm == 0 ? 1 : sd / sm);
	}

	/**
	 * 
	 * @Title getEuclidDistance
	 * @Class testRecommend
	 * @return double
	 * @param pm1
	 * @param pm2
	 * @return
	 * @Description获取两个用户之间的欧几里得距离,距离越小越好
	 * 
	 * @Date 2013-9-4
	 */

	public static double getEuclidDistance(Map<String, Integer> pm1, Map<String, Integer> pm2) {
		double totalscore = 0.0;
		for (Entry<String, Integer> test : pm1.entrySet()) {
			String key = test.getKey();
			Integer a1 = pm1.get(key);
			Integer b1 = pm2.get(key);
			if (a1 != null && b1 != null) {
				double a = Math.pow(a1 - b1, 2);
				totalscore += Math.abs(a);
			}
		}
		return Math.sqrt(totalscore);
	}

	/**
	 * 
	 * @Title getRecommend
	 * @Class testRecommend
	 * @return String
	 * @param simUserObjMap
	 * @param simUserSimMap
	 * @return
	 * @Description根据相关系数得到推荐物品
	 * 
	 * 
	 */
	
	
	public static String getRecommend(Map<String, Map<String, Integer>> simUserObjMap,
			Map<String, Double> simUserSimMap) {
		// 用户
		Map<String, Double> objScoreMap = new HashMap<String, Double>();
		/*		一夜惊喜		环太平洋		变形金刚
		 * P1	3			4			3	
		 * P2	5			1			2
		 * P3	2			5			5
		 * 
		 * /
		// 两个矩阵进行预测计算
		/*	P1		P2		P3		P4		
		P5	0.43	0.66	0.918	0.36*/
		// 遍历那组用户的电影评分数据
		for (Entry<String, Map<String, Integer>> simUserEn : simUserObjMap.entrySet()) {
			// 获取用户的名字
			String user = simUserEn.getKey();
			// 获取对应用户的相关系数   如P1  此时相关系数为0.43
			double sim = simUserSimMap.get(user);
			// 用户各个对三部电影的评分
			for (Entry<String, Integer> simObjEn : simUserEn.getValue().entrySet()) {
				// 获得如 P1用户对一夜惊喜的评分然后用P5对P1的相关系数计算	3*0.43
				double objScore = sim * simObjEn.getValue();
				// P1 P2 P3  此时为P1
				String objName = simObjEn.getKey();
				// 往表填数据 如果key对应的value为空 则直接填入
				 /*P1	1.23+****+****+****
				  *P2	****+****+****+****
				  *P3	****+****+****+****
				  * */
				if (objScoreMap.get(objName) == null) {
					objScoreMap.put(objName, objScore);   	 
															
				} else {
					//如果不为空  则叠加原数据 
					double totalScore = objScoreMap.get(objName);
					objScoreMap.put(objName, totalScore + objScore);
				}
			}
		}
		 /*P1	1.23+****+****+****
		  *P2	****+****+****+****
		  *P3	****+****+****+****
		  * 
		  * */
		// 排序
		List<Entry<String, Double>> enList = new ArrayList<Entry<String, Double>>(objScoreMap.entrySet());
		Collections.sort(enList, new Comparator<Map.Entry<String, Double>>() {
			public int compare(Map.Entry<String, Double> o1, Map.Entry<String, Double> o2) {
				Double a = o1.getValue() - o2.getValue();
				if (a == 0) {
					return 0;
				} else if (a > 0) {
					return 1;
				} else {
					return -1;
				}
			}
		});
		// 返回Top N N =1
		return enList.get(enList.size() - 1).getKey();
	}
}
```
