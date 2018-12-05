>

## 基于User的协调过滤算法

>UserCF：User  Collaboration   Filter，基于用户的协同过滤.

#### 核心思想：

在一个在线推荐系统中，当用户A需要个性化推荐时，系统先找到和他有相似兴趣的其它用户，然后把这些用户喜欢的、而用户A没有听说过的物品推荐给A，这种方法称为基于用户的协同过滤算法。

![](https://i.imgur.com/lElnYGg.png)

图中，user1购买了 1 2 3 4 user2购买了2 user3购买了2 3 ，相比于user2他更合user1相似，于是将user1购买的而user3没有购买的东西推荐给user3.

* 从中可以看出，这个算法主要包括两步：

	* 找到和目标用户兴趣相似的用户集合——计算两个用户的兴趣相似度

	* 找到这个集合中的用户喜欢的，且目标用户没有听说过的物品推荐给目标用户。

#### 具体实现

**计算用户间相识度求最佳**

>CF的常用方法有三种，分别是欧式距离法、皮尔逊相关系数法、余弦相似度法。

1. 皮尔逊相关系数
皮尔逊相关系统是比欧几里德距离更加复杂的可以判断人们兴趣相似度的一种方法。它在数据不是很规范时，会倾向于给出更好的结果。
假定有两个变量X、Y，那么两变量间的皮尔逊相关系数可通过以下公式计算：
![](https://i.imgur.com/yo3zvX8.png)

其中E是数学期望，cov表示协方差，N表示变量取值的个数。
2. 欧几里德距离
假定两个用户X、Y，均为n维向量，表示用户对n个商品的评分，那么X与Y的欧几里德距离就是：


![](https://i.imgur.com/qsRi45d.png)


多维欧几里德距离公式

数值越小则代表相似度越高，但是对于不同的n，计算出来的距离不便于控制，所以需要进行如下转换：

![](https://i.imgur.com/8MUhe2C.png)


相似度公式

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


用户u对物品i感兴趣的程度

其中，p(u,i)表示用户u对物品i的感兴趣程度，S(u,k)表示和用户u兴趣最接近的K个用户，N(i)表示对物品i有过行为的用户集合，Wuv表示用户u和用户v的兴趣相似度，Rvi表示用户v对物品i的兴趣.


## Java实现

```

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

		Map<String, Double> simUserSimMap = new HashMap<String, Double>();

		String output1 = "皮尔逊相关系数:", output2 = "欧几里得距离:";
		// 遍历userMap里面的数据
		for (Entry<String, Map<String, Integer>> userPerfEn : userPerfMap.entrySet()) {
			String userName = userPerfEn.getKey();
			if (!"p5".equals(userName)) {
				// 计算p5用户与其他用户的相关系数，越大越好
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

		// 输出相关系数与结果
		System.out.println(output1);
		System.out.println(output2);
		// 创建一组新用户喜好 map
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
		// 参数一 新的一组用户的爱好     参数二 各用户相关系数
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
	 * 
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
	 * 
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

		Map<String, Double> objScoreMap = new HashMap<String, Double>();
		
		// 遍历那组新用户
		for (Entry<String, Map<String, Integer>> simUserEn : simUserObjMap.entrySet()) {
			// 获取用户的名字
			String user = simUserEn.getKey();
			// 获取对应用户的相关系数
			double sim = simUserSimMap.get(user);
			// 用户各个物品喜好权重遍历
			for (Entry<String, Integer> simObjEn : simUserEn.getValue().entrySet()) {
				
				double objScore = sim * simObjEn.getValue();
				//   A B C D E 
				String objName = simObjEn.getKey();
				// 排除目标用户买过的东西
				if (objScoreMap.get(objName) == null) {
					objScoreMap.put(objName, objScore);
				} else {
					double totalScore = objScoreMap.get(objName);
					objScoreMap.put(objName, totalScore + objScore);
				}
			}
		}
		
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
		return enList.get(enList.size() - 1).getKey();
	}
	}
	```
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


