# 												学习报告

## 专业方面

### 		1.学会了如何控制“敌人”追击玩家

```c#
private GameObject player;
Vector3 playerDir = (player.transform.position - transform.position).normalized;
```

### 		2.学会了如何添加buff

​				添加“碰撞后给予敌人一个反弹作用”的buff （√）

``` c#
private Rigidbody rb;
public float speed;
rb = GetComponent<Rigidbody>();  
float forwardInput = Input.GetAxis("Vertical");
rb.AddForce(focalPoint.transform.forward * speed * forwardInput);
```



### 		3.学会控制敌人波数并逐渐增加敌人数量来达到增加游戏难度

``` c#
public GameObject enemyPrefab;
public int enemyCount;
public GameObject powerUpPrefab;
private int waveNumber = 1;
private Vector3 randomPos;
void Update()
    {
        enemyCount = FindObjectsOfType<Enemy>().Length;

        if (enemyCount == 0)
        {
            SpawnEnemy(waveNumber);
            waveNumber++;
            Instantiate(powerUpPrefab,RandomPos(),powerUpPrefab.transform.rotation);
        }
    }

    public Vector3 RandomPos()
    {
        float spawnPosX = Random.Range(-spawnRange,spawnRange);
        float spawnPosZ = Random.Range(-spawnRange,spawnRange);
    
        Vector3 randomPos = new Vector3(spawnPosX,0,spawnPosZ);
    
        return randomPos;
    }

    void SpawnEnemy(int wave)
    {
        for (int i = 0; i < wave; i++)
        {
            Instantiate(enemyPrefab,RandomPos(),Quaternion.identity);
        }
    }
```

emmmmmmmmmm,复制完代码后，看看自己的代码，好有收获。同时发现这段代码还有着生成PowerUp（提供buff）的效果。

## 	4.好家伙



![好东西](D:\Huawei Share\OneHop\IMG_20201108_111836.jpg)



​				在书架成功捕获“秘籍”，成功给了我新的思路。

![好东西](D:\Huawei Share\OneHop\IMG_20201108_112448.jpg)

由原来的

``` c#
if (Input.GetKeyDown(KeyCode.Space))
        {
            //发射披萨
            Instantiate(food,transform.position,food.transform.rotation);
        }
```

变为

``` c#
if (Input.GetMouseButton(0))
        {
            //发射披萨
            Instantiate(food,transform.position,food.transform.rotation);
        }
```

然后发现在游戏里，一直按着鼠标左键的话，并不能达到预期的目的（一直按着鼠标左键一直发射披萨）。

上网搜索：

![好东西](D:\Huawei Share\OneHop\Screenshot_20201108_112511.jpg)



改为：

``` c#
 if (Input.GetMouseButtonUp(0))
        {
            //发射披萨
            Instantiate(food,transform.position,food.transform.rotation);
        }
```

然后还是不能达到目的，哭唧唧。