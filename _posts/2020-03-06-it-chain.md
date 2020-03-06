---
layout: post
title:  오픈소스 프라이빗 블록체인 코어 개발 프로젝트
date:   2020-03-06 23:28:00 +0800
categories: history
tag: history 
sitemap :
  changefreq : daily
  priority : 1.0

---

이번 포스트에서는 제가 오픈소스 프라이빗 블록체인 코어 개발 프로젝트(이하 [it-chain](https://github.com/DE-labtory/it-chain))에 참여했던 경험을 적겠습니다. 며칠 전에 개발동아리에서 서버 개발했던 경험을 썼는데, 내친김에 이전 경험들을 빨리 다 써버리려고 합니다.

**시기는 2018년 3월 쯤 ~ 2018년 11월 30일 입니다.** 

제가 지금 알고있는 것의 대부분은 이 프로젝트에서 배운 것이라고 해도 과언이 아닐만큼 이 때 많이 배웠습니다. 그만큼 최대한 열심히 적어보겠습니다. 제가 it-chain에 참여하게 된 시발점은 넥스터즈 페이스북 페이지에 올라온 아래 글을 본 것이었습니다.

![그림1](https://dl.dropbox.com/s/figv0sgu3kwng9x/%EC%A4%80%EB%B2%94%ED%8E%98%EC%9D%B4%EC%8A%A4%EB%B6%81%EA%B8%80.png)

저는 이 글을 보자마자 바로 페이스북 메시지를 보냈습니다. 혹시나 사람이 꽉 차서 못들어갈까봐 급하게 보냈던 기억이 나네요. 사실 it-chain 프로젝트는 제가 이전 [개발동아리에서 경험한 첫 서버개발](https://p-vibe.github.io/2020/03/01/history05)을 적을 때 말씀드렸던 동아리 내 다른 프로젝트 중 하나였습니다. OT 때 [준범](https://github.com/junbeomlee)이가 블록체인을 만들자는 아이디어를 냈었는데, 굉장히 멋지다고 생각했지만 도저히 엄두가 나지 않아 이 프로젝트를 기웃거리지도 않았었습니다. 이 때 it-chain 팀의 비하인드 스토리를 말씀드리자면, 이 쪽 팀도 인기가 별로 없었습니다. 개발자가 2명인가 왔었던 걸로 기억나고, 디자이너 3명이 왔었던 것 같습니다. 개발자가 너무 부족해 팀 내 디자이너가 다른 개발자를 꼬셔 총 4명의 개발자와 3명의 디자이너가 프로젝트를 했을거에요. 웃긴 게 준범이가 OT 전 날 '너무 많이 같이 하고 싶다고 하면 어떡하지...? 마음 약해서 쉽게 못 내치는데...' 라고 걱정했는데 실제로는 너무 인기가 없어서 준범이가 굉장히 실망했었습니다...^^; 저는 동아리 활동 중에 it-chain팀이 열심히 하는 걸 보며 재밌어 보이기도 하고, 블록체인을 자기들이 만든다는 게 너무 멋있고, 애초에 제가 동아리를 들어간 이유가 잘하는 개발자를 만나 배우고 싶은 것이었는데 동아리 활동기간 동안 혼자 공부했던 것이 아쉬워 it-chain팀을 부러워했습니다. 그런 상태였는데 저런 글을 봤으니, 바로 지원합니다.

![그림2](https://dl.dropbox.com/s/apngr8fmb8pis91/%EC%A4%80%EB%B2%94%ED%8E%98%EB%A9%94.png)

혹시나 해서 찾아봤는데 그 때 메시지가 남아있네요... 아마 메일도 보냈을 거예요.

준범이가 얘기한대로 문서 읽기를 시작합니다. 그리고 다른 서적들도 찾아가며 조금씩 블록체인 공부를 했습니다. 구현에 대한 문서도 있었던 것 같은데, 그것보단 프로젝트를 소개하는 문서를 주로 봤습니다. 이 문서도 이해가 잘 안돼서 여러번 읽었는데, 블록체인 책이나 강의보다는 훨씬 쉬워서 이 글을 읽고 조금은 이해한 느낌을 받았었습니다. 이 글이 16장 정도 되다보니, 다 올릴 순 없네요. 이 때 쓰여진 내용이 전부 다 맞는 건 아닐 수도 있겠다는 생각이 드는 게 바로 밑 블록체인 설명에 속도가 빠르다는 부분이 의아하긴 하네요. 제가 생각하는 것과 다른 맥락을 갖고 있거나, 어쩌면 이 때는 지식을 쌓는 중이라 조금 틀린 내용이 들어갔을 수도 있을 것 같습니다. 

![그림3](https://dl.dropbox.com/s/ed65vepd6h4j2gp/%EC%9E%87%EC%B2%B4%EC%9D%B8%EC%B2%AB%EB%AC%B8%EC%84%9C.png)

그리고 [이 책](http://www.yes24.com/Product/Goods/67090202) 을 읽다가 도저히 무슨 말인지 모르겠어 잠만 잤고... 오히려 [블로그 글](http://homoefficio.github.io/2017/11/19/%EB%B8%94%EB%A1%9D%EC%B2%B4%EC%9D%B8-%ED%95%9C-%EB%B2%88%EC%97%90-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0/), [유튜브 강의](https://www.youtube.com/watch?v=pPXZs_KjJv8&list=PLlSZlNj22M7QmH8701dVKF2RRExdVhphF&index=1) 등이 처음에 대략적으로 이해하기에 도움이 됐습니다. 그리고 3월 11일에 첫 모임을 가졌었는데요. 그 때 카이스트 박사과정이신 분이 블록체인을 개략적으로 설명해주시는 데 정말 명강의였습니다. 블록체인을 바라보는 관점이 분산 데이터베이스의 일종이라고 보는 시선과 인터넷의 진화 버전이라고 보는 시선으로 나뉜다고 얘기해주셨고, 블록을 생성하기 위해 합의가 필요하다는 것, 합의 중 pow와 pbft에 대한 개략적인 설명, 스마트 컨트랙트에 대한 개략적인 설명 등을 해주셨습니다. 이 후 준범이가 잇체인 프로젝트에 대해서 개략적으로 설명해줬는데요. 이 때 가장 기억나는 말이 **"우리의 목표는 개인의 성장이다"** 라는 말이었어요. 저한테는 꽤 멋지게 들렸습니다. 그리고 어떠어떠한 파트가 있고, 자기가 하고 싶은 걸 하라고 했습니다. 그 때 저는 blockchain 컴포넌트랑 consensus였나 peer가 관심이 간다고 했었는데, 결국 처음부터 끝까지 blockchain 컴포넌트만 하긴 했습니다...^^;;

제 첫 커밋은 제가 문서 읽으면서 오타 수정한 거였습니다. 오타 수정과 더불어, 한 번도 풀리퀘를 날려본 적이 없어서 풀리퀘 날려볼 생각으로 푸시했었는데 이 때는 커밋 기준이 약해서인지 머지됐었네요. 그리고 제가 [코드적으로 처음으로 커밋한 부분](https://github.com/DE-labtory/it-chain/commit/12ae5f97d5bd8c58be336f696bbd304c7d37ddd0)은 json 파일에서 설정값을 가져와, 초기 블록(GenesisBlock)을 생성하는 것이었습니다. 준범이가 되게 잘 이끌어줬던 게, 참여자의 수준보다 조금 더 어려운 과제를 내주고 그걸 깨면서 성장하게끔 도와줬습니다. 그 커밋의 주요 코드는 다음과 같습니다.

```go
// block.go
func CreateGenesisBlock() (*block.DefaultBlock, error) {
	byteValue, err := service.ConfigFromJson("GenesisBlockConfig.json")
	if err != nil {
		return nil, err
	}

	var GenesisBlock *block.DefaultBlock
	json.Unmarshal(byteValue, &GenesisBlock)
	GenesisBlock.Header.TimeStamp = time.Now()
	return GenesisBlock, nil
}
```

```go
//blockConfig.go
func ConfigFromJson(filename string) ([]uint8, error) {

	enginePath := build.Default.GOPATH + "/src/github.com/it-chain/it-chain-Engine/"
	folderPath := viper.GetString("genesisblock.defaultPath")+"/"
	filePath := enginePath + folderPath + filename
	jsonFile, err := os.Open(filePath)
	defer jsonFile.Close()
	if err != nil {
		return nil, err
	}

	byteValue, err := ioutil.ReadAll(jsonFile)
	if err != nil {
		return nil, err
	}
	return byteValue, nil
}
```



그리고 이게 제 인생 첫 테스트 코드입니다.

```go
//block_test.go
func TestCreateGenesisBlock(t *testing.T) {
	GenesisBlock, err := CreateGenesisBlock()
	assert.NoError(t, err)
	assert.Equal(t, uint64(0), GenesisBlock.Header.Height)
	assert.Equal(t, "", GenesisBlock.Header.PreviousHash)
	assert.Equal(t, "", GenesisBlock.Header.Version)
	assert.Equal(t, "", GenesisBlock.Header.MerkleTreeRootHash)
	assert.Equal(t, time.Now().String()[:19], GenesisBlock.Header.TimeStamp.String()[:19])
	assert.Equal(t, "Genesis", GenesisBlock.Header.CreatorID)
	assert.Equal(t, make([]byte, 0), GenesisBlock.Header.Signature)
	assert.Equal(t, "", GenesisBlock.Header.BlockHash)
	assert.Equal(t, 0, GenesisBlock.Header.MerkleTreeHeight)
	assert.Equal(t, 0, GenesisBlock.Header.TransactionCount)
	assert.Equal(t, make([][]byte, 0), GenesisBlock.Proof)
	assert.Equal(t, make([]*tx.DefaultTransaction, 0), GenesisBlock.Transactions)
}
```

사실 쉬운 코드이지만, 제 기억에는 오래남아있네요. 그 이전 커밋은 오타 수정이라... 커밋이라고 하기 좀 그랬거든요. 이게 저한테는 의미있는 첫 커밋이었습니다. 이 때 준범이가 되게 칭찬을 많이 해줬던 것 같아요.

아래는 제가 잇체인 참여 초기에 지난 프로젝트 프론트엔드 개발자와 대화한 내용인데요. 초기인데도 아주 만족하면서 참여했습니다...^^;;

![그림4](https://dl.dropbox.com/s/okusnurvko0hswj/%ED%94%84%EB%A1%A0%ED%8A%B8%EA%B0%9C%EB%B0%9C%EC%9E%90%EC%99%80%EB%8C%80%ED%99%941.jpeg)

![그림5](https://dl.dropbox.com/s/uf79j3q2sguhrd4/%ED%94%84%EB%A1%A0%ED%8A%B8%EA%B0%9C%EB%B0%9C%EC%9E%90%EC%99%80%EB%8C%80%ED%99%942.jpeg)

위 대화에 나온 것처럼 3월부터 조금씩 참여하면서 고 언어와 블록체인의 개략적인 내용을 공부하긴 했지만, 취업준비때문에 많이 참여를 못했습니다. 당시에는 삼성sds를 정말 가고싶었는데, 제가 인적성을 진짜 못해서 인적성 준비를 하느라 시간이 많이 부족했습니다. 겨우 인적성을 통과하고 면접에 갔는데, 기술 면접과 창의성 면접에서는 진짜 감점 요인이 하나도 없었다고 생각할만큼 잘 본것 같고 칭찬도 많이 들었는데(문과인데 열심히 한 것 같다는 말을 많이 들었습니다) 임원 면접에서 제가 긴장을 많이했었습니다. 이게 당시 면접 보고나서 했던 대화인 것 같은데 안망한 줄 알고 있네요...!

![그림6](https://dl.dropbox.com/s/hxhfqqwtxmo2i6l/%EC%82%BC%EC%84%B1%EB%A9%B4%EC%A0%91%ED%9B%84.jpeg)

솔직히 임원면접 빼고 다른 면접들은 정말 잘봤다고 생각해서 붙을 줄 알았는데 5월 29일에 불합격 통보를 받습니다. 이전까지는 진짜 조금씩 참여하면서 드문드문 커밋하는 수준이었던 것 같은데 이 때부터 본격적으로 잇체인 프로젝트에 집중합니다. 이후부터 블록체인 컴포넌트 코드의 대부분을 보고 손댄 것 같아요. 이 때 꽤 빠르게 성장했었는데, 블록체인 말고 다른 컴포넌트에 이미 다른 사람이 짜논 코드를 따라하면서 블록체인 컴포넌트 코드를 짰어요. 저는 그냥 어떻게 하는 지 모르겠으니까 다른 코드 참고하면서 짰던 건데, 이 때 많이 성장한 것 같아요. 다른 사람이 왜 그렇게 짰는 진 모르겠지만, 나도 비슷하게 짜버릇 했고 조금씩 "얜 왜 이렇게 짰을까?"라고 생각하면서 그 사람의 코드를 배운 것 같아요. 돌이켜 생각했을 땐 이 때가 많이 성장했던 때라고 기억하지만, 당시에는 별로 성장하는 느낌이 안들었어요. 오히려 절망했죠. 이전 넥스터즈 서버개발 프로젝트를 할 때는 하면 할수록 자신감이 팍팍 상승하는 느낌이었는데, 여기선 오히려 하면 할수록 제 부족함을 느꼈어요. 같이 프로젝트를 하는 동료들이 다 잘하고 제가 제일 못하다 보니까 "역시 프로그래밍이라는 게 아무나 하는 게 아닌가?" 라는 생각을 많이 했어요. 코딩하는 순간 순간은 재밌고 좋은데, 앞이 막막했어요. 그 당시에 "나는 코딩에 재능이 없다"라고 생각했습니다. 제가 며칠 간 고민하고 삽질했는데도 안되던 걸 주변 동료는 순식간에 처리하는 걸 보니 자연스럽게 그런 생각이 들었죠. 그 즈음 준범이랑 같이 지하철에서 했던 대화가 기억나는데요. 준범이는 실력으로 일짱먹는, 최고 개발자가 되고 싶다는 얘기를 했어요. 저는 거기에 "나는 잘하는 건 안바라고 그냥 평범한, 중간정도 하는 개발자 하고 싶어" 라고 했었습니다. 요즘엔 잘하고 싶다는 욕심이 많은데, 1년 정도만에 마인드가 많이 바뀌었다는 느낌이 드네요. 어쨌든 그 땐 미래에 대한 불안감이 컸고, 주변에 나보다 열심히 하지도 않는 것 같은데 컴퓨터 전공자나 복수전공자들이 번듯한 대기업에 취업하는 걸 보면서 역시 비전공자는 안되나 하는 생각도 했습니다. 드문드문 지쳐서 그냥 아무데나 좋으니까 취업하고 싶다라는 생각도 들었어요. 다행히 코딩 하는 것 자체는 재미있고, 동료들의 말이 힘이 돼서 꾸준히 공부했던 것 같습니다. 제가 너무 못하는 것 같다고 투정부리니까 준범이가 자신감 그래프라는 것이 있는데, 원래 성장할 때 오히려 자신감이 많이 떨어진다고 했습니다. 그리고 자기가 볼 땐 빠르게 성장하고 있는 것 같고, 지금처럼 꾸준히 하면 1~2년 뒤에는 네이버 개발자 뚜까팰 수 있다고 했습니다. 힘들 때마다 이런 말들이 위로가 돼서 포기하지 않았던 것 같아요.

다시 프로젝트 내용으로 돌아가자면, 저는 블록의 생성, 저장, 블록체인 동기화, 다른 컴포넌트와의 소통(이벤트를 발생시키거나 소비하는 부분)을 구현했습니다. 제가 가장 신경 쓴 부분은 블록체인 동기화 부분인데요. 준범이가 왜 블록체인 동기화가 필요한 지, 어떠어떠한 시나리오가 있는 지 대략적으로만 알려주고 구체적인 내용은 안알려줬습니다. 주형이와 토론하면서 우리끼리 시나리오를 짰고, 시나리오대로 동기화하기 위해 블록의 상태를 정의해줬습니다. 아래는 함께 고민한 내용을 바탕으로 제가 작성한 [블록체인 컴포넌트의 문서](https://github.com/DE-labtory/it-chain/blob/develop/blockchain/README.md) 중 일부입니다.

----------------------------------------

## Key Concepts

it-chain은 Blockchain Component를 customize 할 수 있도록 block 저장 및 조회를 [yggdrasill](https://github.com/it-chain/yggdrasill)에 위임한다. Blockchain Component는 yggdrasill에서 정의한 interface에 맞게 구조체를 구현한다면 yggdrasill에 block을 저장, 조회할 수 있다. yggdraill에 연속적으로 저장된 block들이 blockchain이다.

#### Block State

Blockchain Component는 맡은 역할을 원활히 수행하기 위해 Block의 상태를 다음과 같이 구분하였다.

| Block State | Description                                    |
| ----------- | ---------------------------------------------- |
| Created     | 합의되지 않았고, blockchain에 저장되지 않았다. |
| Staged      | 합의되었지만, blockchain에 저장되지 않았다.    |
| Committed   | 합의되었고, blockchain에 저장되었다.           |



## Blockchain Synchronize

1. 동기화(Synchronize)는 특정 노드의 블록 체인을 네트워크 내 임의의 노드의 블록 체인과 동일하게 만드는 과정을 의미한다. 즉 동기화(Synchronize) 과정을 통해 특정 노드는 모든 블록에 대하여 대표값(Seal), 이전 블록의 대표값(PrevSeal), 트랜잭션 모음(TxList), 트랜잭션 대표값(TxSeal), 블록 생성 시각(TimeStamp), 생성자(Creator), 블록 체인의 길이(Height) 등의 블록 체인과 관련된 모든 정보들을 다른 노드의 것과 동일화한다.
2. 동기화(Synchronize)는 **확인(Check)**, **구축(Construct), 재구축(PostConstruct)** 의 과정을 거친다.
3. **확인(Check)** 은 특정 노드의 블록 체인이 동기화가 필요한 상태인지를 점검한다. **확인(Check)** 의 과정은 임의의 노드에게 Blockchain 길이와 lastSeal을 받아와서 자신의 블록 체인 정보가 같은 지 비교하여, 동기화가 필요한 상태인지 점검한다(SyncedCheck). 이미 동기화가 완료된 상태라면, 동기화(Synchronize) 과정을 중단한다. 그렇지 않을 경우, **구축(Construct)** 을 수행한다.
4. **구축(Construct)** 은 임의의 노드에게 블록 정보를 요청하여, 응답받고, 응답받은 블록을 블록 체인에 저장하는 과정을 순차적으로 반복함으로써 수행된다.
5. 블록 요청은 특정 노드의 블록 체인 길이(Height)를 활용해, 임의의 노드에 블록을 요청함으로써 수행된다. 특정 노드가 새로 참여하는 노드일 경우 임의의 노드의 블록 체인 내 최초 블록부터 마지막 블록까지 요청하고, 기존에 참여중이던 노드일 경우 보유 중인 블록 체인 내 마지막 블록의 다음 블록부터 임의의 노드의 블록 체인 내 마지막 블록까지 요청한다.
6. 임의의 노드의 모든 블록이 특정 노드의 블록체인에 저장되면 **구축(Constrcut)**이 완료된다.
7. 특정 노드는 **구축(Construct)** 의 진행 중에 새롭게 합의되는 블록을 블록 임시 저장소(BlockPool)에 보관한다. **구축(Construct)** 이 완료되고 나면, 블록 임시 저장소에 블록이 보관되어 있는 지 확인한다(PoolCheck). 보관중인 블록이 있다면, **재구축(PostConstruct)**을 수행한다.
8. **재구축(PostConstruct)** 은 이미 **구축(Construct)** 된 블록 체인에 블록 임시 저장소(BlockFool)에 보관중인 블록들을 부수적으로 추가하는 것을 의미한다. **재구축(PostConstrcut)** 을 수행하고 나면, 동기화(Synchronize) 과정이 모두 완료된다.

## Scenario

본 시나리오는 Blockchain Component와 관련된 흐름을 다른 Component와 연동하여 간략하게 보여준다.

1. 노드 A는 Genesis.conf 파일을 알맞게 수정 후 GenesisBlock을 생성하여 P2P 네트워크를 시작한다. 이 시나리오에서는 노드 A를 리더노드라고 가정한다.
2. 노드 B는 Genesis.conf 파일을 노드 A의 것과 동일하게 수정 후 GenesisBlock을 생성하여 노드 A가 만든 P2P 네트워크에 참여한다.
3. Client는 네트워크의 프록시 서버에 transaction을 요청하고, 프록시 서버는 transaction을 네트워크 내 노드들에 분배한다.
4. 노드 B의 TxPool Component에 transaction이 쌓이면 리더인 노드 A에 transaction을 전송한다.
5. 노드 A의 TxPool에 transaction이 쌓이면 transation모음과 blockchain 내 마지막 block 정보를 토대로 새로운 block을 생성한다.
6. 노드 A는 생성한 block을 Consensus Component에 넘겨 노드 B와 합의한다.
7. block이 합의를 마치고 나면 노드 A, 노드 B 모두 해당 block을 blockchain에 저장한다. 저장한 block은 event에 담아 발행하여 다른 Component에 blockchain에 block이 저장되었다는 사실을 알린다.
8. 노드 C는 GenesisBlock을 생성하여 노드 A와 노드 B가 있는 P2P 네트워크에 참여한다.
9. 노드 C는 네트워크 내 임의의 노드와 blockchain을 동기화한다.
10. 만약 9의 과정 중 3~6의 과정이 진행될 경우, 노드 C는 합의된 block의 상태를 staged로 변경하고 BlockPool에 임시 저장한다. 9의 과정이 완료되면 임시 저장한 block의 상태를 committed로 변경하고 blockchain에 저장 및 event 발행한다.

------------------------------

준범이가 처음에는 블록체인 동기화를 다른 통신으로 하려고 했습니다. 그런데 제가 그걸 잘 몰라서 그냥 http통신으로 동기화합니다...

중간에 [공개sw개발자센터](https://www.oss.kr/kosslab) 라는 곳에서 세미나를 하기도 합니다. 리더인 준범이가 당시 공개sw개발자센터에서 일했기 때문에 그 쪽에서 발표를 해달라고 요청했었어요. 당시 다른 핵심 멤버들이 개인 사정 때문에 발표를 할 수가 없어 제가 it-chain에 대해 발표했습니다. 사실 이 때 영광스럽긴 했지만 잇체인팀을 대표한다는 게 부담스럽고 떨렸어요. 세미나를 보러 오는 사람 대부분이 현업 개발자일텐데, 그러면 분명히 나보다 잘할 것이라는 생각이 들어서 잔뜩 쫄아있었죠. 발표 준비를 아주 열심히 했는데, 외우다시피 한 내용을 긴장한 나머지 너무 빠르게 말해서 듣는 사람들 입장에서 무슨 말인지 못 알아들었을 것 같아요. 그리고 발표가 끝난 후 혹여나 제가 모르는 질문이 들어올까 겁이 나서 급하게 마무리지었습니다. 돌이켜 생각해보면 오신 분들이 아무리 실력이 저보다 뛰어나도 잇체인에 대해서는 제가 더 많이 알 수밖에 없었고 모르면 그냥 모른다고 하면 됐는데 좀 더 자신감을 가지고 발표할걸 하는 아쉬움이 남았습니다.

![그림7](https://dl.dropbox.com/s/anavxmta8dynfus/%EA%B3%B5%EA%B0%9Csw%EA%B0%9C%EB%B0%9C%EC%84%BC%ED%84%B0%EC%84%B8%EB%AF%B8%EB%82%98%ED%8F%AC%EC%8A%A4%ED%84%B0.png)

![그림8](https://dl.dropbox.com/s/kbj347gzh66rgbx/%EA%B3%B5%EA%B0%9Csw%EA%B0%9C%EB%B0%9C%EC%9E%90%EC%84%BC%ED%84%B0%EB%B0%9C%ED%91%9C%EC%9E%A5%EB%A9%B4.png)

그리고 아마 8월 쯤, 잇체인 팀이 약간의 쉬는 시간을 가지면서 사이드 프로젝트를 했었습니다. 준범이가 예전에 [마이크로소프트웨어](https://www.imaso.co.kr/)라는 잡지사에 글을 기고한 적이 있어서 그 쪽 기자분과 아는 사이였는데, 그 기자분이 한번 더 글을 써주기를 요청했습니다. 우리가 당시 msa와 ddd를 적용하고 있었기 때문에 msa, ddd, eventsourcing을 주제로 [예제코드](https://github.com/hea9549/msa-ddd-eventsourcing)를 짜고 [마이크로소프트웨어 394호](https://www.imaso.co.kr/archives/3939)에 글을 썼습니다.

![그림9](https://dl.dropbox.com/s/g3etmpw463m7dvb/%EC%9E%A1%EC%A7%80.png)

이 때 잡지 출간된 걸 찍은 것인데, 제가 가진 건 이 사진 뿐입니다...ㅜㅜ. 이 때 준범이가 한 번에 받고 멤버들한테 나눠줬는데, 저는 타이밍이 안맞아 못받다가 흐지부지됐고 시중에 나와있는 거 나중에 사야지 하고 사는 걸 미뤄두고 있었는데 절판됐네요. 제 사진이 들어간 책인데 갖지 못해 좀 아쉽습니다... 그리고 제 이름이 맨 위에 올라가있는데 아무 의미 없고, 그냥 자기 소개 작성 선착순이었습니다.

재밌는 게 제주도 여행 가서 놀면서 예제 코드 짜고 잡지 쓰자고 해서 다 같이 제주도에 갔습니다. 이 때 사진들을 좀 올려놓겠습니다.

제주로 출발하는 사진입니다. 제가 찍자고 졸라서 다 같이 찍었어요.

![제주1](https://dl.dropbox.com/s/q1q6j0po1jr16g4/%EC%A0%9C%EC%A3%BC3.jpeg)

첫 날 도착해서 밤에 먹은 딱새우입니다. 너무 맛있었고 이 날 술을 너무 많이 마셔서 다음날부터 술이 잘 안들어갔습니다...

![제주2](https://dl.dropbox.com/s/a14yrk5vcw6kcs3/%EC%A0%9C%EC%A3%BC5.jpeg)



코딩도 많이 했습니다.

![제주4](https://dl.dropbox.com/s/m6jh6pj83twa4mg/%EC%A0%9C%EC%A3%BC7.jpeg)



이 사진 보면 뭔가 토론하는 것 같은데 토론이라기보다는 해성이한테 엄청 물어보는 중일 것입니다.

![제주3](https://dl.dropbox.com/s/erq868zhop4lgkd/%EC%A0%9C%EC%A3%BC6.jpeg)

![제주5](https://dl.dropbox.com/s/1ech8wwi3ewab4v/%EC%A0%9C%EC%A3%BC11.png)



![그림10](https://dl.dropbox.com/s/wwoy547bqo9a3dl/%EC%A0%9C%EC%A3%BC%EB%8F%84%EC%97%AC%ED%96%89%EC%BD%94%EB%94%A9.png)



술은 거의 매일 밤 마셨는데 며칠간 누적되니까 더 못먹겠더라고요... 우정해장국집이라고(우진 아님), 제 인생 국밥집이었는데요. 점심에 갔는데 너무 맛있어서 술 시켰다가 한잔 먹고 토할 뻔 했습니다.

![제주6](https://dl.dropbox.com/s/hngeb98wsb2m24e/%EC%A0%9C%EC%A3%BC4.jpeg)

준범이네 연구실 박사님이 제주도에 계셔서 비싼 것도 얻어먹었습니다. 이 날이었나... 준범이가 갑자기 2시간 뒤에 병특 회사 서류 제출 마감이라고해서 다 같이 자소서 썼던 게 생각나네요.

![제주12](https://dl.dropbox.com/s/2anycnddv74fd0o/%EC%A0%9C%EC%A3%BC2.jpeg)



임주현 도착. 주현이는 운전해야 했기 때문에 주현이 빼고 다 맥주 마셨습니다. 주현이가 오고 서울에 일 있는 친구들은 먼저 복귀했습니다. 그런데 남은 사람들은 태풍에 갖혀서 예정보다 오래 제주도에 머물렀어요.

![제주7](https://dl.dropbox.com/s/f16g9u3zpw70sh9/%EC%A0%9C%EC%A3%BC10.jpeg)



주현이가 풀빌라 쏴서 난생 처음으로 풀빌라도 가봤네요.

![제주8](https://dl.dropbox.com/s/9d5s9mao7fkd20x/%EC%A0%9C%EC%A3%BC8.jpeg)



굉장히 설정 느낌이 나는 설정 사진입니다.

![제주10](https://dl.dropbox.com/s/vcc61sb3vrjg1hm/%EC%A0%9C%EC%A3%BC9.jpeg)

유명한 카페 가서 찍은건데 태풍 때문에 하늘이 아주 흐립니다. 목욕탕도 갔던 게 생각나네요.

![제주11](https://dl.dropbox.com/s/sies5bg9xgthpm0/%EC%A0%9C%EC%A3%BC1.jpeg)



마지막 날 공항가는 길인데 자세히 보면 무지개가 보입니다.

![제주13](https://dl.dropbox.com/s/jjzn17mqjorj64x/%EC%A0%9C%EC%A3%BC12.jpeg)



사진 한두개 넣으려고 했는데 찾다보니까 재밌어서 굉장히 삼천포로 빠졌네요. 다시 돌아와서, 저는 이 때 온라인 쇼핑몰 중 주문 서비스를 구현했고, 글에는 ddd 개념 중 anti-corruption layer에 대해서 썼습니다. 제가 쓴 부분만 옮겨볼게요.

---------------------

[중제] Anti-Corruption Layer
손상 방지(Anti-Corruption) 레이어는 바운디드 컨텍스트 2개를 연결한다. 손상 방지 레이어를 사용하는 이유는 두 개의 하위 시스템 사이에 발생하는 과도한 의존성을 제거하기 위함이다. 여러 하위 시스템이 직접적인 의존 관계를 가지면 하위 시스템 1개가 수정될 때 의존된 하위 시스템이 심각하게 손상돼 많은 코드를 수정해야 한다. 다른 하위 시스템을 이해하지 못하는 개발자는 시스템의 변화에 큰 곤란을 겪을 수 있다. 
반면 손상 방지 레이어를 통해 하위 시스템이 간접적으로 의존되면, 손상 방지 레이어 수정만으로 전체 시스템을 유지할 수 있다. 즉 손상 방지 레이어를 통해 외부 시스템의 변화에 따른 영향을 최소화할 수 있다. 
-------------------- <박스>

손상 방지 레이어의 적절한 활용- 레거시 시스템과 새 시스템 통합을 유지, 관리해야 하는 경우.- 두 개의 시스템이 다른 의미를 갖지만 통신이 필요한 경우.------------------

쇼핑몰 애플리케이션 중 ‘Order’ 도메인에서 손상 방지 레이어를 적용한 예시를 소개한다.
‘Order’ 서비스 인프라 레이어는 HTTP 통신 등으로 다른 서비스와 협력해 ‘Order’ 서비스 애플리케이션, 도메인 레이어로 해결할 수 없는 문제를 지원한다. 예를 들어 사용자가 주문할 때 ‘Order’ 서비스는 ‘Order’ 객체를 데이터베이스에 저장하기 전 사용자가 원하는 제품 재고가 남았는지 확인해야 한다. 재고는 ‘Product’ 서비스에서 담당하기 때문에 ‘Order’ 서비스는 ‘Product’ 서비스와 협력할 필요가 있다.
------------------ <코드4> ‘Product’ 서비스에 HTTP 요청을 보내 원하는 정보를 얻기 위한 코드

// Infra Layer@Service
public class HttpProductAdapter implements ProductAdapter {

  @Autowired
  @Qualifier("productRestTemplate")
  RestTemplate productRestTemplate;

  public int getStockByProductId(String productId) {
    ProductInfoDTO productInfo = productRestTemplate.getForObject("/products/"+productId,ProductInfoDTO.class);
    if (productInfo.getProductId() == null ) {
      throw new IllegalArgumentException("productInfo invalid");
    }
    return productInfo.getStock();
  }
}------------------

<코드4>의 ‘ProductAdapter’는 ‘Product’ 서비스와 HTTP 통신해 제품 정보를 가져온다. 이 코드에서 ‘ProductInfoDTO’는 ‘Order 서비스에서 필요한 제품 정보를 ‘Product 서비스'로부터 전달받기 위해 정의한 객체다.
------------------ <코드5> 인프라 레이어에서 ‘Product’ 서비스로부터 제품 정보를 받기 위해 정의한 객체 코드 @AllArgsConstructor
@Data
public class ProductInfoDTO {
  private String productId;
  private String buyerId;
  private int stock;
}------------------

‘ProductInfoDTO’는 ‘Product’ 서비스와 관련이 깊은 객체다. 그 때문에 이 객체가 ‘Order’ 서비스의 애플리케이션, 도메인 레이어에서 직접 사용되면 ‘Product’ 서비스의 변화가 ‘Order’ 서비스에 큰 영향을 미칠 수 있다. 이는 ‘Order’ 서비스가 ‘Product’ 서비스 변화에 민감해진다는 것을 뜻한다.
이를 막기 위해 <코드4>의 ‘ProductAdapter’에는 손상 방지 레이어가 적용돼 있다. 예제 코드에서는 ‘ProductAdapter’가 ‘ProductInfoDTO’에서 ‘Order’ 서비스에서 필요한 ‘stock’ 정보만을 리턴해준다. 이는 ‘Order’ 서비스가 직접 ‘Product’ 서비스와 연관이 깊은 객체 ‘ProductInfoDTO’를 사용하는 것을 막는다. 이로써 ‘Order’ 서비스의 다른 코드는 ‘ProductInfoDTO’에 대한 의존성을 제거할 수 있다.
<코드6>을 통해 ‘Product’ 서비스로부터 얻은 ‘stock’ 정보는 <코드7>처럼 활용된다.
--------------------- <코드6> 인프라 레이어에서 ‘ProductAdapter’를 이용해, ‘Product’ 서비스와 협력하기 위한 코드// Infra Layer@Service
public class ProductServiceImpl implements ProductService {

  @Autowired
  ProductAdapter productAdapter;

  public int getStockByProductId(String productId) {

​    return productAdapter.getStockByProductId(productId);
  }
}---------------------
--------------------- <코드7> 애플리케이션 레이어에서 ‘Product’ 서비스에서 가져온 값을 실제로 사용하는 코드// Application Layer
@AutowiredproductService productService;
int stock = productService.getStockByProductId(productId);---------------------

‘Order’ 서비스의 애플리케이션 레이어에서는 ‘Product’ 서비스와 직접 관련된 객체 ‘ProductInfoDTO’를 사용하거나 호출하지 않는다. 덕분에 ‘Product’ 서비스에서 변화가 생기더라도 ‘Order’ 서비스는 코드 레벨에서 깨지지 않고 쉽게 유지 보수할 수 있다.

-----------------



제주도에 다녀오고 나서는 다시 잇체인 작업에 몰두합니다. 잇체인팀은 [컨트리뷰톤](https://www.oss.kr/notice/show/ee15de47-7adc-48a5-b4bc-039ba04192af)에 참여중이었는데, 막바지 작업에 한창 중일 때  제가 transaction 컴포넌트의 이벤트 발생, 소비 부분을 작성중에 rebase 실수로 코드를 다 날려먹어 식겁했던 적이 있습니다. 가까스로 복구해서 4개의 노드를 띄워 합의 후 블록을 생성하는 시나리오를 동영상으로 찍어 컨트리뷰톤 담당 기관에 제출했습니다. 우리가 생각했던대로 블록이 생성되는 걸 보고 아주 기뻤습니다. 컨트리뷰톤에서는 은상을 수상했습니다. 사실 우리끼리 우리가 무조건 금상이라고 김칫국을 마신 상태였기 때문에 기쁘면서 살짝 아쉬웠습니다.

![컨트리뷰톤](https://dl.dropbox.com/s/0fu01jpzu0dzjlb/%EC%A0%9C%EC%A3%BC14.jpeg)

이 때 받은 상금으로 랍스타 먹었습니다.

![랍스타](https://dl.dropbox.com/s/kklgrx1avl3f1ha/%EC%A0%9C%EC%A3%BC13.jpeg)

이날 대부분의 기여자분들이 모여 찍은 기념사진입니다. 제 2018년의 8할이라고 할 수 있는 it-chain 프로젝트가 이걸로 마무리됐습니다.

![수상기념사진](https://dl.dropbox.com/s/tki3l1q907ldyb1/%EC%88%98%EC%83%81%EA%B8%B0%EB%85%90%EC%82%AC%EC%A7%84.png)



it-chain 때는 이전에 회고했던 다른 프로젝트들이랑은 분량 차이가 커서 쓰기 버거울 것이라고 생각했는데 쓰고 나니 후련하네요. 행복한 경험이었습니다.

이제 현재와 싱크를 맞추기까지 얼마 안남았습니다. 잇체인 프로젝트를 위해 유예했던 넥스터즈에 복귀해서 했던 프로젝트와 현재까지 진행중인 블록체인 기반 게임 프로젝트가 있는데요. 블록체인 기반 게임 프로젝트는 제대로 쓰려면 완성 후 써야할 것 같아서 두 프로젝트를 아예 2019년 프로젝트로 합칠까 생각중입니다. 2019년에는 제가 회사생활을 해서 개발적으로 많은 걸 하진 않았거든요.



긴 글 읽어주셔서 감사합니다.