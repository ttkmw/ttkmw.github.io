---
layout: post
title:  우아한 redis 정리
date:   2020-07-06 01:43:00 +0800
categories: 책/강의
tag: db
sitemap :
  changefreq : daily
  priority : 1.0
---

# 우아한 Redis 정리

이 포스트는 [이 세미나](https://www.youtube.com/watch?v=mPB2CZiAkKM)를 보고 정리한 것입니다. 사실 redis를 처음으로 써보면서, 초보자를 위한 redis 사용 팁 등을 알고자 영상을 봤는데 제가 이해하기엔 어려운 강의였습니다. 실무자들이 사용하면서 생길만한 이슈들을 얘기해주시고, 그에 대한 해결책 위주로 설명해주셨습니다. 그래서 내용을 이해하지 못한 것도 많이 있습니다. 아직 이해 못했다고 하더라도 나중에 보고 '대강 이러이러한 내용이었더라' 정도는 남기려고 게시글을 작성합니다. 발표자의 블로그는 [여기](https://charsyam.wordpress.com/)입니다.

## 1. Redis 개요

redis는 리모트 프로세스에 있는 in-memory database이다.

**redis의 자료구조**  

- Strings(키,밸류), set, sorted-set, hashes, list
- Hyperloglog, bitmap, geospecial index
- Stream

Cache는 나중에 올 요청에 대한 결과를 미리 저장해두었다가 빠르게 서비스해주는 것을 의미한다. Disk - Memory - L3 cache - L2 cache - L1 cache - core 뒤로 갈수록 용량은 줄고 속도는 빨라진다.

일반적인 애플리케이션은 client - web server - db 구조를 가진다. db 안에는 많은 데이터들이 있고 이 때 주로 디스크에 내용이 저장된다(db도 자기의 캐시를 가지긴 함). 

**캐시 구조 1 : look aside cache**

client - web server - db / cache

1. web server는 데이터가 존재하는지 cache를 먼저 확인한다.
2. cache에 데이터가 있으면 cache에서 데이터를 가져온다.
3. cache에 데이터가 없으면 db에서 가져온다.
4. db에서 읽어온 데이터를 cache에 저장한다.

**캐시 구조 2 : write back**

1. web server는 모든 데이터를 cache에만 저장한다.
2. cache에 특정 시간동안 데이터를 저장한다.
3. cache에 있는 데이터를 db에 저장한다.
4. db에 저장된 데이터를 cache에서 삭제한다.

memory cache를 이용하면 훨씬 빠른 서비스 가능하다.

## 2. 왜 Collection이 중요한가?

1. 개발의 편의성
   랭킹 서버 개발의 사례: 간단한 방법은 db 에 유저의 score를 저장하고 score를 order by로 정렬 후 읽어오는 것이다. 개수가 많아지면 속도가 느려질 수 있(디스크를 사용하므로). 그래서 in-memory 기준으로 랭킹 서버 구현이 필요하다. redis sorted-set 활용하면 된다.

2. 개발의 난이도

   친구 리스트 관리 개발 사례: 친구 리스트를 key/value로 저장한다면, 친구 리스트에 A가 저장되어있고 B와 C를 등록할 때, B, C가 각각 하나의 트랜잭션으로 독립적인 작업이므로 상황에 따라 친구 리스트에 A,B / A,C / A,B,C가 들어갈 수 있다. redis의 자료구조는 atomic하기 때문에 이러한 상황을 피할 수 있다. 

결국 외부의 collection을 잘 이용하면 되기 때문에, 개발 시간을 단축시키고 문제를 줄여주기 때문에 collection이 중요하다.

## 3. redis 사용처

1. remote data store : 여러 대의 서버에서 같은 데이터 스토어를 공유하고 싶을 때 사용한다.
2. 그렇다면 하나의 서버일 때는 필요 없는가(전역변수를 쓰면 되니까)? Redis는 싱글 스레드이고 atomic하기 때문에 이슈가 덜하다. 따라서 쓰는것도 좋다.
3. 주로 많이 쓰는 곳: 인증 토큰 등 저장, 랭킹 보드로 사용, 유저 api limit 등

## 4. Redis collection

1. Strings(키,밸류)
   Set [key] [name], Get [key]

   키를 뭘로 저장할지를 고민해야한다.

2. List
   Lpush [key] [A], Rpush [key] [A] / Lpop [key], Rpop [key]

3. Set
   SADD [key] [value] / SMEMBERS [key] / SISMEMBERS [key] [value]

   예시: 특정 유저를 follow하는 목록을 저장

4. Sorted Set : 유저 랭킹 보드 등으로 활용 가능하다. score가 실수형이기 때문에 조심해야한다(범위때문에).
   ZADD [key] [score] [value]

   ZRANGE [key] [stardindex] [endindex] 

   스코어 값에 따라 정렬 등을 할 수 있다.

5. Hash : key 밑에 sub key가 존재한다(키 밸류 안에 다시 키 밸류를 넣을 수 있게 해준다).

   Hmset [key] [subkey1] [value1] [subkey2] [value2]

   Hgetall [key]

   Hget [key] [subkey]

**collection 주의사항**

1. 하나의 컬렉션에 너무 많은 아이템을 담으면 좋지 않음 : 10,000개 이하(몇천개 수준)을 유지하는 게 좋다.

2. Expire는 Collection의 item 개별로 걸리지 않고 전체 Collection에 대해서만 걸린다. 즉 해당 10,000개의 아이템을 가진 Collection에 expires가 걸려 있다면 그 시간 후에 10,000개 모두 삭제된다.

## 5. redis 운영

**1. 메모리 관리를 잘하자**

(1) 메모리 관리의 중요성 : redis는 인메모리 db이기 때문에 physical memory 이상을 사용하면 문제가 발생한다. 바로 죽는 건 아니고 swap이 있으면 swap 사용한다. 다만 이렇게 되면 디스크를 사용하게 돼서 느려진다. swap이 없으면 죽을 수 있다. Maxmemory를 설정하더라도 이보다 더 사용할 가능성이 크다. 더 큰 문제는 많은 업체가 현재 메모리를 사용해서 swap을 쓰고 있다는 것을 모를 때가 많다.

redis는 메모리 파편화가 발생할 수 있다. 4.x대부터 메모리 파편화를 줄이도록 jemalloc에 힌트를 주는 기능이 들어갔으나, jemalloc 버전에 따라서 다르게 동작할 수 있다. 3.x대 버전의 경우 실제 used memory는 2GB로 보고가 되지만 11GB의 RSS를 사용하는 경우가 자주 발생했다. 한 가지 팁은 다양한 사이즈를 가지는 데이터보다는 유사한 크기의 데이터를 가지는 경우가 유리하다는 것이다.

(2) 메모리가 부족할때는?

cache is cash! :  좀 더 메모리가 많은 장비로 migration 해야한다. 메모리가 빡빡하면 migration 중에 문제가 발생할 수도 있다.

있는 데이터 줄이기. 다만 이미 swap을 사용중이라면, 프로세스를 재시작해야한다. 

(3) 메모리를 줄이기 위한 설정

기본적으로 Hash는 Hash Table을 하나 더 사용하고, Sorted Set은 Skiplist와 Hash Table을 이용하고, Set은 Hash Table을 사용한다. 이러한 자료구조들은 메모리를 많이 사용한다. 이럴 때 Ziplist를 쓰면 속도는 조금 느려지지만 메모리는 적게 쓰게 된다. Ziplist는 선형 탐색을 하는데, 적은 양일 때는 선형 탐색해도 문제 없다. List, hash, sorted set등을 ziplist로 대체하는 설정이 존재한다. 많은 데이터의 경우 속도가 좀 느려질 수 있겠지만, 메모리를 2~30% 줄일 수 있다.

**2. o(n) 명령어는 주의하자**

레디스는 싱글스레드이기 때문에 동시에 처리할 수 있는 요청 개수는 한개다. 레디스의 구조는 Packet으로 하나의 Command가 완성되면 processCommand에서 실제로 실행되는 구조이다. 즉 뒤의 패킷은 쌓일 것이다. 긴 시간을 요하는 명령을 실행하면 안된다. 예를 들어, KEYS, FLUSHALL, FLUSHDB, Delete Collections, Get All Collections 등을 수행하면 안된다. 그렇다면 KEYS는 어떻게 대처할 것인가? scan 명령어로 긴 명령을 짧은 여러번의 명령으로 바꿀 수 있다. Collection의 모든 item을 가져와야 할 때는 어떻게 할 것인가? Collection의 일부만 가져온다거나, 큰 Collection을 작은 여러개의 Collection으로 나눠서 저장한다.

**3. Replication**

Redis는 좋은 게 Replication을 해준다. Replication이라 함은 A라는 서버의 데이터를 B서버도 가지고 있는 것을 말한다(어떤 이슈가 있을 수 있다고 하는데 무슨 말인지 모르겠음)

(1) Replication 설정 과정

1. Secondary에 replicaof or slaveof 명령을 전달한다.

2. Secondary는 Primary에 sync 명령 전달한다.

3. Primary는 현재 메모리 상태를 저장하기 위해 Fork를 한다. 이게 만악의 근원이다.

4. Fork를 하기 때문에 프로세스는 현재 메모리 정보를 disk에 dump한다(메모리도 많이 쓰게 됨).

5. 해당 정보를 secondary에 전달한다.

6. Fork 이후의 데이터를 secondary에 계속 전달한다.

(2) Redis Replication 시 주의할 점

1. Replicatoin 과정에서 fork가 발생하기 때문에 메모리 부족이 발생할 수 있다.

2. redis-cli--rdb 명령은 현재 상태의 메모리 스냅샷을 가져오므로 같은 문제를 발생시킨다.

3. AWS나 클라우드의 redis는 좀 다르게 구현되어서 좀 더 해당 부분이 안정적이다.

4. 많은 대수의 Redis 서버가 Replica를 두고 있다면, 네트워크 이슈나 사람의 작업으로 동시에 replication이 재시도 되도록 하면 문제가 발생할 수 있다. 예를 들어, 같은 네트워크 안에서 30GB를 쓰는 Redis Master 100대 정도가 Replication을 동시에 재시작하면 네트워크가 마비될 수 있다.

## 6. 권장 설정 TIP

1. Maxclient 설정 50,000
2. RDB / AOF 설정 off
3. 특정 commands diable. Keys는 어지간하면 diable 하는게 좋다.
4. 전체 장애의 90% 이상이 KEYS, SAVE설정(1분 안에 키가 10,000가 바뀌었으면 지금 메모리를 dump해 이런 설정)을 사용해서 발생한다.

## 7. Redis 데이터 분산

redis를 쓰기 때문에 생기는 문제는 아니고 데이터를 캐시로 쓰냐, persistent한 스토어로 쓰냐에 따라 우아한 redis가 될지 오픈더헬게이트가될 지 나뉜다.

데이터 분산 방법이 Application이냐, Redis Cluster를 쓰냐에 따라 나뉜다.

(몸으로 서버가 꽉 찼을 때의 문제 상황을 보여주심)

서버가 두 대였다가 꽉차서 한 대를 늘리면, 데이터 re-balancing이 필요하다. 서버 한대가 죽으면? 리밸런싱이 또 필요하다... 장애에 취약한 방식이다. - 이에 대해 2의 배수(x2)로 서버를 늘리면 괜찮다. 물론 한번 늘릴려면 나중에는 엄청 많이 늘려야하는 문제가 생긴다.
개선한 방식으로, 서버를 해싱한다. 서버 한 대가 죽으면, 그 서버에 해당하는 데이터만 리밸런싱 하면 된다. 이게 consitent hashing이다. 키에 대한 해시값이 일정하기 때문에 가능한 방식이다.

## 8. 샤딩

'데이터를 어떻게 나눌 것인가?'는 '데이터를 어떻게 찾을 것인가?'와 같은 고민이다. 상황마다 샤딩 전략이 달라진다. 가장 쉬운 방법이 range(id 1~1000번은 1번 서버로 배정 등. 문제는 이벤트 기간에 회원가입이 증가하는 등의 이유로 하나의 서버에만 데이터가 몰릴 수 있다는 것이다. )를 사용하는 방법이다.

## 9. Redis Cluster

(이 부분 무슨 말인지 모르겠음)

Hash 기반으로 Slot 16384로 구분

- Hash 알고리즘은 CRC16을 사용
- Slot = crc16(key) % 16384
- Key가 Key{hashkey} 패턴이면 실제 crc16에 hashkey가 사용된다.
- 특정 redis 서버는 이 slot range를 가지고 있고, 데이터 migration은 이 slot 단위의 데이터를 다른 서버로 전달하게 됨

Redis Cluster의 장점

1. 자체적인 Primary, Secondary Failover

2. Slot 단위의 데이터 관리

Redis Cluster의 단점

1. 메모리 사용량이 더 많음

2. Migration 자체는 관리자가 시점을 결정해야 함

3. Library 구현이 필요함

## 10. Redis Failover

1. Coordinator 기반 Failover

2. Vip/DNS 기반 Failover

3. Redis Cluster의 사용

## 11. Monitoring

Redis Info를 통한 정보

1. RSS

2. Used Memory

3. Connection 수

4. 초당 처리 요청 수

System

1. CPU

2. Disk

3. Network rx/tx

처리량이 많다면 좀 더 cpu 성능이 좋은 서버로 이전한다.

o(n) 계열의 특정 명령이 많은 경우 Monitor 명령을 통해 특정 패턴을 파악하는 것이 필요하다. Monitor 잘못 쓰면 부하로 해당 서버에 더 큰 문제를 일으킬 수도 있다.

## 12. 결론

기본적으로 Redis는 좋은 툴이다. 메모리를 빡빡하게 쓸 때부터 안좋은 툴로 변한다. Redis as Cache로 쓸 땐 문제가 적게 발생한다. Redis가 문제 생기면 DB로 얼마만큼의 부하가 가는지 확인 필요하다. Redis as Persistent Store로 쓸 때는 관리 이슈가 크게 발생한다. 메모리도 여유있게 잡아야하는 등(마이그레이션 할 때도 문제 없도록)의 대책이 필요하다.