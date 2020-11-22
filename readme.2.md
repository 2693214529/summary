# 学习报告

## 学会了如何随机生成障碍物，并且保证连通

​	原本只是随后生成障碍物的，但为了保证游戏的可玩行，强行加上了保证流通性。嘤嘤嘤，这个代码搞了我将近1给星期，估计是最大的工程量了（希望）。

同时这段代码在我慢慢一点点敲出来的时候学会了好多东西（开心）。

```c#
using System;
using UnityEngine;
using System.Collections.Generic;

public class MapGenerator : MonoBehaviour
{
    public GameObject tilePrefab;
    public Vector2 mapSize;
    public Transform mapHolder;
    [Range(0,1)]public float outlinePercent;
    [Range(0,1)]public float obsPercent;
    public GameObject obsPrefab;
    
    public List<Coord> allTilesCoord = new List<Coord>();

    public Color foregroundColor,backgroundColor;
    public float minObsHeight,maxObsHeight;

    private Queue<Coord> shuffledQueue;

    private Coord mapCenter;
    bool[,] mapObstacles;

    public Vector2 mapMaxSize;
    public GameObject nevMeshObs;
    public GameObject player;

    private void Start ()
    {
        GenerateMap();
        Ins();
    }

    private void Ins()
    {
        Instantiate(player,new Vector3(-mapSize.x / 2 + 0.5f + mapCenter.x,0,-mapSize.y / 2 + 0.5f + mapCenter.y),Quaternion.identity);
    }

    private void GenerateMap() 
    {
        for (int i = 0; i < mapSize.x; i++) //瓦片生成
        {
            for (int j = 0; j < mapSize.y; j++)
            {
                Vector3 newPos = new Vector3(-mapSize.x / 2 + 0.5f + i,0,-mapSize.y / 2 + 0.5f + j);
                GameObject spawnTile = Instantiate(tilePrefab,newPos,Quaternion.Euler(90,0,0));
                spawnTile.transform.SetParent(mapHolder);
                spawnTile.transform.localScale *= (1 - outlinePercent);

                allTilesCoord.Add(new Coord(i,j));
            }
        }

        shuffledQueue = new Queue<Coord>(Utilities.ShuffleCoords(allTilesCoord.ToArray()));

        int obsCount = (int)(mapSize.x * mapSize.y * obsPercent);
        mapCenter = new Coord((int)mapSize.x / 2,(int)mapSize.y /2);
        mapObstacles = new bool[(int)mapSize.x,(int)mapSize.y];

        int currentObsCount = 0;

        for (int i = 0; i < obsCount; i++) //生成障碍物
        {
            Coord randomCoord = GetRandomCoord();

            mapObstacles[randomCoord.x,randomCoord.y] = true;
            currentObsCount++;

            if (randomCoord != mapCenter && MapIsFullAccessible(mapObstacles,currentObsCount))
            {
                float obsHeight = Mathf.Lerp(minObsHeight, maxObsHeight, UnityEngine.Random.Range(0f,1f));

                Vector3 newPos = new Vector3(-mapSize.x / 2 + 0.5f + randomCoord.x,obsHeight / 2,-mapSize.y / 2 + 0.5f + randomCoord.y);
                GameObject spawnObs = Instantiate(obsPrefab,newPos,Quaternion.identity);    
                spawnObs.transform.SetParent(mapHolder);

                spawnObs.transform.localScale = new Vector3(1 -outlinePercent,obsHeight,1 - outlinePercent);
                #region 

                MeshRenderer meshRender = spawnObs.GetComponent<MeshRenderer>();
                Material material = meshRender.material;

                float colorPencent = randomCoord.y / mapSize.y;
                material.color = Color.Lerp(foregroundColor,backgroundColor,colorPencent);

                #endregion
            }
            else
            {
                mapObstacles[randomCoord.x,randomCoord.y] = false;
                currentObsCount--;
            }

        }

        GameObject nevMeshObsForward = Instantiate(nevMeshObs,Vector3.forward * (mapSize.y + mapMaxSize.y) / 4,Quaternion.identity);
        nevMeshObsForward.transform.localScale = (new Vector3(mapSize.x, 5,(mapMaxSize.y / 2 - mapSize.y /2)));
        
        GameObject nevMeshObsBack = Instantiate(nevMeshObs,Vector3.back * (mapSize.y + mapMaxSize.y) / 4,Quaternion.identity);
        nevMeshObsBack.transform.localScale = new Vector3(mapSize.x, 5,(mapMaxSize.y / 2 - mapSize.y /2));
        
        GameObject nevMeshObsLeft = Instantiate(nevMeshObs,Vector3.left * (mapSize.x + mapMaxSize.x) / 4,Quaternion.identity);
        nevMeshObsLeft.transform.localScale = new Vector3((mapMaxSize.x / 2- mapSize.x / 2), 5,mapSize.y);

        GameObject nevMeshObsRight = Instantiate(nevMeshObs,Vector3.right * (mapSize.x + mapMaxSize.x) / 4,Quaternion.identity);
        nevMeshObsRight.transform.localScale = new Vector3((mapMaxSize.x / 2- mapSize.x / 2), 5,mapSize.y);

    }

    private bool MapIsFullAccessible(bool[,] _mapObstacles,int _currentObsCount)
    {
        bool[,] mapFlags = new bool[_mapObstacles.GetLength(0),_mapObstacles.GetLength(1)];

        Queue<Coord> queue = new Queue<Coord>();
        queue.Enqueue(mapCenter);
        mapFlags[mapCenter.x,mapCenter.y] = true;

        int accessableCount = 1;

        while (queue.Count > 0)
        {
            Coord currentTile = queue.Dequeue();

            for (int x = -1; x <= 1 ;x++)
            {
                for (int y = -1; y <= 1; y++)
                {
                    int neightborX = currentTile.x + x;
                    int neightborY = currentTile.y + y;

                    if (x == 0 || y == 0)
                    {
                        if (neightborX >= 0 && neightborX < _mapObstacles.GetLength(0)
                         && neightborY >= 0 && neightborY < _mapObstacles.GetLength(1))
                        {
                            if (!mapFlags[neightborX,neightborY] && !_mapObstacles[neightborX,neightborY])
                            {
                                mapFlags[neightborX,neightborY] = true;
                                accessableCount++;
                                queue.Enqueue(new Coord(neightborX,neightborY));
                            }
                        }
                    }
                }
            }
        }


        int obsTargetCount = (int)(mapSize.x * mapSize.y - _currentObsCount);
        return accessableCount == obsTargetCount;

    }

    private Coord GetRandomCoord()
    {
        Coord randomCoord = shuffledQueue.Dequeue();
        shuffledQueue.Enqueue(randomCoord);
        return randomCoord;

    }
}

[System.Serializable]
public struct Coord
{
    public int x;
    public int y;
    public Coord(int _x,int _y)
    {
        this.x = _x;
        this.y = _y;
    }
    public static bool operator != (Coord _c1,Coord _c2)
    {
        return !(_c1 == _c2);
    }
    public static bool operator == (Coord _c1,Coord _c2)
    {
        return (_c1.x == _c2.x) && (_c1.y == _c2.y);
    }
}
```

​	好家伙，刚刚复制完这段代码，自己都看不下去。