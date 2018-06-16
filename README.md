###### ※ 이 블록체인 협의 알고리즘은 공개 알고리즘으로 아직 이론 단계입니다.<br/><br/>블록체인을 연구하고 개발하는 분들과 함께 토론하고 검증하며 알고리즘이 수정 발전되길 바랍니다.<br/><br/>비선형 수치를 잘 다루시는 분들이 참여하시면 수학적으로 증명 가능한 완전한 알고리즘이 만들어지리라 생각합니다.<br/><br/>그리고 이 문서의 [영문화 작업](https://github.com/ninanoo/PoR)을 도와주실 분들을 찾습니다. 해당 내용을 [이슈](https://github.com/ninanoo/PoR---Korean-Version/issues/1)로 등록해 두었습니다.

<br/>

# PoR ( Proof of Relevancy )

### 관련성에 의한 증명 ( 관련성에 의해 결정되는 블록체인 합의 알고리즘 )

관련성에 의한 증명(proof of relevancy)은 인접 블록 간에 각 블록의 소유정보로부터 구해진 관련성(relevancy)에 의해 다음 블록이 정해지는 블록체인 합의 알고리즘(blockchain consensus algorithm) 입니다.
관련성은 작업증명(proof of work)의 작업이나 지분증명(proof of stake)의 지분으로부터 만들어 질 수도 있습니다.
여기서는 각 블록의 소유키를 관련성을 만드는 소유정보의 예로 사용합니다.
비트코인(Bitcoin)과 같은 해시체인을 사용하지만 비트코인NG(Bitcoin-NG)와 같은 이중 체인 구조를 갖습니다.
이중 체인은 서로 교차하여 해시값이 연결된 키블록체인(key block chain)과 인증블록체인(auth block chain)으로 구성됩니다.
이중 체인과 함께 관련성, 역방향 관련성(reverse relevancy), 인증블록 생성을 위한 임계값(threshold value) 등의 특성으로 인해 원장에 대한 인증은 결정적(deterministic)으로 동작합니다.
원장에 대한 인증이 결정적으로 이루어지므로 이중지불(double spending)과 피니어택(finney attack) 등이 발생하지 않습니다.
51% 공격(51% attack)과 지역화(localization) 문제에 대해 비트코인보다 더 강한 내성을 갖습니다.
네트워크의 규모에 따라 머클트리(merkle tree)의 운용 없이 단위 원장에 대한 개별 인증블록들을 생성할 수 있을 만큼 고속화 처리가 가능합니다.

<br/>

## 블록과 체인의 구조 ( Structure of Blocks and Chains )

![blockAndChainDiagram](blockAndChainDiagram.png?raw=true "blockAndChainDiagram")

키블록체인에 대부분은 인증블록 생성을 마친 블록들과 인증블록을 생성하기 위한 블록으로 확정됐었으나 악의적인 목적 또는 네트워크 장애로 인해 하위 블록에 의해 제외 처리된 블록들로 구성되고 뒷부분은 인증블록 생성을 위한 후보블록들로 구성됩니다.
후보블록들 중 가장 먼저 생성된 후보블록이 리더블록이 됩니다.
모든 키블록에는 순서번호와 생성 당시 마지막으로 확정됐던 인증블록의 순서번호와 관련성의 산출에 필요한 소유정보들이 포함되고 이들을 앞 키블록의 해시값과 생성 당시 마지막으로 확정됐던 인증블록의 해시값과 조합하여 새로 만들어진 해시값이 포함됩니다.
인증블록체인은 원장들의 인증 정보가 들어있는 원장인증블록들과 키블록체인에서 제외된 키블록에 대한 제외 정보가 들어있는 제외인증블록들로 구성됩니다.
원장인증블록들은 인증이 확정된 블록과 가장 최근 생성된 확정되지 않은 하나의 블록으로 나누어집니다.
원장인증블록에는 사용된 키블록의 순서번호와 인증할 원장들의 인증 정보가 포함되고 이들 정보들을 앞 인증블록의 해시값과 사용된 키블록의 해시값과 조합하여 새로 만들어진 해시값이 포함됩니다.
제외인증블록에는 제외된 키블록의 순서번호와 이를 제외시킨 키블록의 순서번호와 관련 제외 정보가 포함되고 이들 정보들을 앞 인증블록의 해시값, 제외된 키블록의 해시값 그리고 이를 제외시킨 키블록의 해시값과 조합하여 새로 만들어진 해시값이 포함됩니다.
키블록체인과 인증블록체인의 순서번호는 같은 순서로 쌍을 이루어 증가합니다.
인증블록체인의 마지막 블록의 순서번호보다 큰 순서번호를 갖는 키블록은 모두 후보블록입니다.

<br/>

## 관련성과 합의 방법 ( Relevancy and Consensus Mechanism )

![consensusDiagram1](consensusDiagram1.png?raw=true "consensusDiagram1")

새로 추가될 후보블록들은 네트워크에서 끊임없는 확산 수렴 과정을 거치며 앞 블록과 가장 관련성이 높은 블록으로 합의가 됩니다.
어느 한 블록 자체의 관련성(self relevancy)은 앞 블록의 해시값과 해당 블록의 공개키(public key)를 첫 비트부터 비트순으로 비교하였을 때 연속해서 일치하는 비트들의 수를 통해서 생성 됩니다.
네트워크에서의 불필요한 경쟁 상황을 줄이기 위해 비트코인과 유사한 난이도(difficulty)를 갖습니다.
난이도는 일치하여야 하는 최소 비트수로 작업을 증명하기 위한 목적은 아닙니다.
단순히 공개키를 사용하는 것은 블록의 위치에 상관없이 변하지 않는 값을 사용하는 것으로 키 풀을 운용하는 방법 등으로 불필요한 자원 소모를 유발할 수 있습니다.
실제로는 앞 블록의 해시값과 해당 블록의 소유정보를 조합하여 새로 생성된 값으로 비교합니다.
앞 블록의 해시값과 해당 블록의 공개키 등의 소유정보를 조합하여 해당 블록의 해시값을 구해 앞 블록의 해시값과 비교하는 방법이 있습니다.
이는 모든 블록의 해시값의 앞부분이 같은 값으로 유지되게 하고 서비스 별로 별도의 체인으로 운용할 때 각 체인을 구분하는 식별자로 사용될 수 있습니다.

![consensusDiagram2](consensusDiagram2.png?raw=true "consensusDiagram2")

실제 블록의 관련성은 해당 블록을 포함하면서 해당 블록의 하위 블록들에 관련성이 누적되어 정해집니다.
하위 블록들에 관련성은 체인에 추가된 순서에 따라 지수 분포로 배율이 정해집니다.
이는 상위 블록일수록 교체되는 횟수가 적어지게 만들어 체인이 결정적으로 동작하게 만듭니다.
후보블록은 체인의 끝이 아니더라도 관련성이 기존 블록보다 더 높으면 해당 블록을 대체할 수 있고 기존 블록의 하위 블록들도 함께 버려집니다.
전체 관련성은 점점 더 높아지면서 블록의 수는 더 적어지도록 계속해서 합의가 이루어집니다.
체인의 앞부분부터 충분한 관련성을 갖는 블록들로 채워지게 되고 실제 트랜잭션은 체인의 뒷부분의 블록들에서만 일어납니다.

![consensusDiagram3](consensusDiagram3.png?raw=true "consensusDiagram3")

비트코인의 작업증명에서는 체인에 분기가 발생하는 것을 충돌 상황으로 간주하고 가장 긴 체인을 선택합니다.
관련성에 의한 증명에서는 체인에 분기가 발생하는 것을 알고리즘이 동작하는 과정으로 간주합니다.
분기가 발생한 블록의 관련성을 비교하여 더 높은 값을 갖는 체인이 선택됩니다.
불필요한 작은 단위의 분기를 피하기 위해 난이도를 만족하는 `m` 값이 사용됩니다.
협의 과정에 상당 부분을 차지할 만큼 체인에 분기는 자주 발생하나 대부분 체인의 뒷부분의 블록들에서만 일어납니다.
이를 보증하기 위해 인증블록 생성을 위한 임계값을 충분히 높도록 설정합니다.
이는 체인이 운용되는 전체 네트워크의 물리적 환경과 요구되는 성능치에 의해 결정됩니다.

아래는 어느 한 블록의 관련성을 산출하는 식입니다.

![relevancyFormula](relevancyFormula.png?raw=true "relevancyFormula")
```
r      : relevancy
c      : number of following blocks including the target block ( c > 0 )
n      : sequence number of candidate blocks starting with 0 ( n < c )
m of n : number of consecutive bits with the same value as the previous hash value starting from the beginning
a      : relevancy factor ( a > 1 )
```

`r`은 `n = 0`인 `0`번째 블록의 누적 관련성으로 `c - 1`개의 하위 블록들이 뒤따르고 있습니다.
`2 ^ m`은 각 `n`번째 하위 블록의 자체 관련성입니다.
`a ^ (-n / c)`은 각 `n`번째 하위 블록의 관련성 배율(relevancy ratio)입니다.
누적 관련성을 계산하려는 `0`번째 블록의 관련성 배율은 언제나 `1`로서 뒤따르는 블록들이 없는 경우 누적 관련성은 자체 관련성과 같습니다.
각 하위 블록의 자체 관련성에 각자의 관련성 배율이 곱해지고 이들을 모두 더한 값이 어느 한 블록의 관련성이 됩니다.
체인의 모든 후보들록들은 각각 자신을 `0`번째 블록으로 하여 위의 산출식을 통해 관련성을 계산합니다.

아래는 `(2 ^ 16) ^ (-n / 10000)`의 관련성 배율 그래프입니다.

![relevancyRatio1](relevancyRatio1.png?raw=true "relevancyRatio1")

위 그래프는 관련성 인자(relevancy factor) `a`로 `2 ^ 16`을 사용했고 현재 `10000 - 1`개의 하위 블록들이 뒤따르고 있습니다.
절반에 해당하는 `5000`번째 이전에 추가됐던 블록들부터 의미 있는 값을 갖기 시작해 1%에 해당하는 `100`번째 이전에 추가됐던 블록들은 `0.9` 이상의 값을 갖습니다.

아래는 관련성 인자로 `2 ^ 16`을 사용하면서 `c`가 각각 10, 100, 10000일 때의 관련성 배율 그래프들입니다.
`n / c`의 특성을 확인할 수 있는 그래프로 각 하위 블록들의 관련성 배율은 전체 하위 블록수 대비 각 블록의 위치에 대한 일정한 비율값을 갖습니다.

![relevancyRatio2](relevancyRatio2.png?raw=true "relevancyRatio2")

블록의 수가 `10`일 때 `9`번째 블록의 관련성 배율은 `0`에 가까운 값입니다.
이 블록에 하위 블록들이 추가되어 블록의 수가 `100`이 됐을 때 `9`번째 블록의 관련성 배율은 대략 `0.4`에 가까운 값을 갖습니다.
다시 하위 블록들이 추가되어 블록의 수가 `10000`이 됐을 때는 `1`에 가까운 값을 갖습니다.
어느 한 하위 블록에 또 다른 하위 블록들이 추가될수록 해당 블록의 곡선에서의 위치는 왼쪽 방향으로 계속 이동하여 관련성 배율은 지수 증가하게 됩니다.

다음은 `a ^ (-n / 10000)`의 관련성 배율 그래프로 위쪽의 곡선부터 각각 `2 ^ 2`, `2 ^ 4`, `2 ^ 8`, `2 ^ 16`, `2 ^ 32`의 관련성 인자를 사용합니다.

![relevancyRatio3](relevancyRatio3.png?raw=true "relevancyRatio3")

그래프에서 볼 수 있듯이 관련성 인자가 커지면 지수 감소하는 관련성 배율의 감소폭도 커집니다.
그로 인해 큰 관련성 인자를 사용할수록 누적 관련성은 작아지나 체인은 더 결정적으로 동작합니다.

<br/>

## 임계값과 관련성 효율비 ( Threshold Value and Relevancy Efficiency Ratio )

첫 번째 후보블록인 리더블록은 다음 인증블록을 생성하기 위한 블록으로 확정되기 위해 임계값을 갖습니다.
리더블록의 관련성이 임계값을 만족할 때까지 하위 블록들의 교체와 추가가 계속해서 이루어지며 관련성은 점점 더 높아집니다.
리더블록의 관련성이 임계값에 다다르면 해당 블록은 인증블록의 생성을 시작하고 하위 블록의 추가는 더 이상 발생하지 않으나 관련성이 더 높은 경우 교체는 계속해서 이루어집니다.
리더블록에 의해 생성된 인증블록의 배포가 시작되면 다음 후보블록이 리더블록이 되고 새 리더블록의 관련성을 기준으로 하위 블록들의 교체와 추가를 이어갑니다.

단순하게 `r / c`는 관련성 산출에 포함된 블록들의 수 대비 해당 관련성으로 관련성 효율비(relevancy efficiency ratio)입니다.
네트워크에서는 이 값이 더 높아지도록 계속해서 협의가 이루어집니다.
이 값이 크면 각 블록들이 충분한 자체 관련성을 가지고 있는 것으로 임계값 산출을 위한 지표가 될 수도 있습니다.

다음은 관련성 그래프를 모의실험하여 임계값을 추정하기 위한 관련성 산출식입니다.

![expectedRelevancyFormula](expectedRelevancyFormula.png?raw=true "expectedRelevancyFormula")
```
r      : relevancy
c      : number of following blocks including the target block ( c > 0 )
n      : sequence number of candidate blocks starting with 0 ( n < c )
m of n : number of consecutive bits except the difficulty bits ( m > 0 )
d      : difficulty bits ( d > 0 )
a      : relevancy factor ( a > 1 )
```

각 블록의 자체 관련성은 난이도를 포함하면서 첫 번째 블록의 `m` 비트부터 마지막 블록의 `m / c` 비트까지 추가된 순서의 균일한 비율로 분포합니다.
상위 블록일수록 긴 시간동안 더 많은 협의를 거치고 생성된 지 얼마 되지 않은 하위 블록일수록 난이도만 만족한 정도일 것이라는 실제 상황을 가정한 것입니다.
그러나 실제 일치하는 비트수와 그 분포는 네트워크 환경과 알고리즘의 기준값들에 의해 정해집니다.
이를 위한 정확한 실험을 위해서는 테스트넷(testnet)을 구성하거나 실제 환경에 근접한 소프트웨어 시뮬레이션이 수행되어야 합니다.

아래는 위의 관련성 산출식에서 `d`가 `8`이고 `m`도 `8`일 때의 관련성 그래프입니다.
위쪽부터 각각 `2 ^ 2`, `2 ^ 4`, `2 ^ 8`, `2 ^ 16`, `2 ^ 32`의 관련성 인자를 사용합니다.

![relevancy1](relevancy1.png?raw=true "relevancy1")

위의 그래프를 통해 각 관련성 인자별로 특정 임계값을 만족하기 위한 하위 블록들의 수를 대략적으로 확인할 수 있습니다.
`2 ^ 16`의 관련성 인자를 사용하는 그래프의 경우 대략 `2 ^ 25`의 관련성을 갖기 위해 `10000`개의 블록이 필요합니다.
새로운 블록이 `2 ^ 25`의 관련성을 갖는 블록을 대체하기 위해서는 자체적으로 `25`개 이상의 일치하는 비트수를 가지고 있어야 합니다.
이는 작업증명의 작업량과 비교될 수 있습니다.
작업증명에서 연속된 `0`을 갖는 해시값을 찾는 노력은 관련성에 의한 증명에서 앞 해시값과 일치하는 해시값을 찾는 노력과 같은 논리로 동작합니다.
작업증명에서는 조건을 만족하는 해시값을 찾기 위해 짧지 않은 시간동안 각 노드의 시스템 자원을 소모합니다.
관련성에 의한 증명에서는 노드들이 소유한 고유한 해시값들 중에 가장 조건을 만족하는 해시값을 찾기 위해 매우 긴 시간동안 네트워크 자원을 소모합니다.
작업증명에서는 해시값을 찾는 전체 시간동안 모든 채굴 노들들에서 균일하게 시스템 자원을 소모하나 관련성에 의한 증명에서는 전체 시간에서 주로 블록이 추가된 직후의 짧은 시간동안만 해당 트랜잭션들이 발생합니다.
각 관련성 인자별로 안전하게 사용할 수 있는 일치하는 비트수들이 정해져야 하고 그에 따라 임계값이 정해져야 합니다.
이를 위해 이미 검증된 작업증명의 보안 속성들에 맞추어 관련성에 의한 증명에서의 각 기준값들이 정해져야 합니다.

아래는 위의 그래프와 같은 난이도를 사용하나 `m`이 `12`일 때의 관련성 그래프입니다.

![relevancy2](relevancy2.png?raw=true "relevancy2")

`2 ^ 16`의 관련성 인자를 사용하는 그래프의 경우 대략 `2 ^ 25`의 관련성을 갖기 위해 `1000`개의 블록이 필요합니다.
일치하는 비트수가 `4`비트 증가했을 때 필요한 블록 수는 `1 / 10`로 줄었습니다.
임계값 만족을 위한 필요 블록수를 줄이기 위해서는 임계값을 줄이거나 더 작은 관련성 인자를 사용하는 방법 등이 있습니다.
그러나 이들은 모두 결정적으로 동작하는 알고리즘의 특성을 약하게 만듭니다.
일치하는 비트수를 올리기 위해 더 큰 난이도를 사용하는 것을 생각할 수 있으나 이는 알고리즘의 기본 동작 방식과 맞지 않습니다.
알고리즘이 정하는 바는 새로운 블록이 체인에 추가되는데 큰 제약을 두지 않습니다.
낮은 난이도를 사용할수록 더 많은 블록들이 더 빠르게 체인에 추가가 되고 이는 리더블록이 더 빠르게 인증블록을 발행할 수 있게 만듭니다.
난이도는 단지 네트워크에서의 소모적인 동작들을 줄이기 위함입니다.

매우 큰 임계값과 관련성 인자 그리고 지수 감소하는 누적값의 사용 등으로 인해 어느 한 지점에서 임계값을 만족하기 위해서는 매우 많은 블록들이 필요합니다.
그로 인해 키블록체인이 네트워크에서 처음 만들어지기 시작할 때 최초의 리더블록은 임계값을 만족하는데 충분한 시간을 가져야 합니다.
그러나 첫 블록 이후의 블록들은 리더블록이 되었을 때 하나의 노드만 추가되어도 임계값이 만족될 수 있습니다.
리더블록이 되는 순간 이미 임계값이 만족되었을 수도 있습니다.
매우 많은 블록들 중에서 하나의 블록만이 관련성 값 산출에서 제외되었기 때문입니다.
이는 인증블록 생성의 고속화 처리를 가능하게 합니다.
머클트리를 사용할 수도 있으나 네트워크의 규모에 따라 하나의 원장에 하나의 블록을 사용할 수 있을 정도로 성능이 나올 수 있습니다.
리더블록들의 소모가 매우 빠를 경우 하위 단에서 충분한 관련성 효율비가 유지되도록 기준값들의 값을 키워야 합니다.

<br/>

## 역방향 관련성과 게임 이론 ( Reverse Relevancy and Game Theory )

악의적인 의도나 네트워크 장애로 인해 임계값을 만족한 리더블록의 원장인증블록 배포가 지연되는 상황이 발생할 수 있습니다.
이를 위해 모든 키블록은 역방향 관련성을 갖습니다.
협의를 위한 관련성은 체인에서 뒤따르는 블록들의 누적값이나 역방향 관련성은 그 반대 방향인 이전 블록들의 누적값으로 뒤따르는 인접한 블록에 의해 계산됩니다.
후보블록들은 자신의 관련성을 산출하는데 사용된 `c` 값을 적용하여 이전 후보블록들의 역방향 관련성을 산출합니다.
임계값을 만족한 블록의 바로 다음 블록은 자신의 관련성과 앞 블록의 역방향 관련성을 비교합니다.
배포된 새 원장인증블록을 받기 전에 자신의 관련성이 임계값을 만족하고 앞 블록의 역방향 관련성보다 자신의 정방향 관련성이 커지면 앞 블록을 제외된 블록으로 처리하고 자신이 원장인증블록을 생성하여 배포합니다.
제외인증블록과 원장인증블록이 함께 배포가 되고 이를 받은 다른 노드들은 관련 정보를 확인한 후 수락하거나 거절합니다.
만약 두 번째 블록에서도 원장인증블록의 배포가 이루어지지 않으면 세 번째 블록은 첫 번째와 두 번째 블록 모두에 대해 같은 방법으로 조건이 만족 되면 자신이 원장인증블록을 발행합니다.
후보블록들 중 앞부분부터 대략 `1 / 10`에 해당하는 블록들은 모두 같은 방법으로 이전 후보블록들을 제외시킬 수 있는 권한을 갖습니다.

역방향 관련성에 누적 계산되는 이전 블록들은 모두 네트워크에서 충분한 확산 수렴 과정을 통해 높은 자체 관련성을 가지고 있는 블록들입니다.
그러나 인증블록 배포가 지연되는 상황이 발생하면 하위 체인의 후보블록들은 추가 없이 교체만으로 관련성을 높이므로 `c` 값은 점점 더 작아져 상대적으로 역방향 관련성은 더 낮아지게 됩니다.
이로 인해 역방향 관련성은 임계값에 다다른 리더블록이 최대한 빠른 시간 내에 인증블록을 발행하도록 촉구합니다.
후보블록들 중 앞부분부터 대략 `1 / 10`에 해당하는 블록들은 모두 같은 상황에 놓이게 됩니다.

제외시킬 수 있는 권한을 갖는 블록들의 정확한 수는 관련성 인자에 의해 정해집니다.
관련성 배율 그래프에서 `0`에 근접하지 않은 의미 있는 배율을 갖는 블록들이 이에 해당합니다.
이는 관련성 배율 곡선의 변화량(derivative)과 관계있습니다.

다음의 그래프는 위에서 살펴본 관련성 인자별 관련성 배율 곡선들과 그들의 변화량에 대한 곡선들을 함께 나타낸 것입니다.

![relevancyRatioDerivative](relevancyRatioDerivative.png?raw=true "relevancyRatioDerivative")

위쪽이 관련성 인자별 관련성 배율 곡선들이고 아래쪽이 각 곡선들의 `n / c`에 대한 변화량 곡선들입니다.

아래는 변화량 산출에 사용된 식입니다.

![relevancyRatioDerivativeFormula](relevancyRatioDerivativeFormula.png?raw=true "relevancyRatioDerivativeFormula")
```
d : derivative
c : number of following blocks including the target block ( c > 0 )
n : sequence number of candidate blocks starting with 0 ( n < c )
a : relevancy factor ( a > 1 )
```

1차적인 변화량에 대한 분석만으로는 아직 명확한 관계가 찾아지지 않아 관련성 인자별로 대략적인 값을 사용합니다.
`2 ^ 16`의 관련성 인자를 사용하는 그래프의 경우 후보블록들 중 앞부분부터 대략 `1 / 10`에 해당하는 블록들을 고려하였으나 실제 구현에서는 전체 네트워크의 물리적 환경이나 요구되는 보안 수준에 맞게 충분히 높도록 설정해야 합니다.

<br/>

## 동기화 ( Synchronization )

분산시스템의 특성상 리더블록에 의한 원장인증블록 배포와 두 번째 후보블록에 의한 제외인증블록 배포 간에 충돌이 발생할 수 있습니다.
대규모 네트워크에서 사용할 경우 알고리즘의 빠른 트랜잭션 처리로 인해 다른 협의 알고리즘들에 비해 더 많은 충돌이 발생할 수도 있습니다.
이는 세 번째 후보블록의 선택에 따르나 알고리즘에서 정하는 바는 다음과 같습니다.
세 번째 후보블록에게 리더블록의 원장인증블록이 먼저 도착하면 세 번째 후보블록은 이를 정상적인 상황으로 간주합니다.
세 번째 후보블록에게 두 번째 후보블록의 제외인증블록이 먼저 도착하면 리더블록은 제외 처리 됩니다.
다른 방법으로는 세 번째 후보블록이 네트워크에 충돌 상황을 알리는 방법도 있습니다.
그러나 이는 또 다른 동기화 문제를 발생시킬 수 있습니다.
역방향 관련성과 마찬가지로 후보블록들 중 앞부분부터 대략 `1 / 10`에 해당하는 블록들은 모두 동기화 문제 처리에 대한 책임이 있습니다.
네트워크의 다른 모든 노드들은 차례로 선택되는 후보블록들의 선택을 따릅니다.

<br/>

## 이중지불 ( Double Spending )

악의적인 리더블록은 연결된 두 개의 서로 다른 네트워크에 각각의 인증블록을 발행해 이중지불을 시도할 수 있습니다.
다음 리더블록은 결정적으로 이미 선출되어 있고 두 인증블록 중 하나를 먼저 받게 됩니다.
해당 블록에 자신의 인증블록을 추가하고 같은 키블록에 의해 동시 발행된 다른 인증블록들은 버려집니다.
운용 환경에 따라 악의적인 행위를 시도한 노드에게 불이익을 줄 수도 있습니다.
비록 결정적으로 동작하기는 하나 인증블록체인의 마지막 인증블록은 아직 인증이 확정되지 않은 블록으로 간주합니다.
어느 한 인증블록에 하나 이상의 인증블록이 뒤따를 때 인증이 확정됩니다.
그러나 이는 매우 빠른 시간 안에 이루어지고 여전히 결정적 알고리즘으로 동작합니다.

하나의 뒤따르는 인증블록 만으로도 확정이 이루어지나 관련성을 이용하여 인증 확정을 위한 임계값을 사용할 수도 있습니다.
추가되는 모든 키블록은 마지막으로 확정된 인증블록의 순서번호와 해시값을 사용합니다.
이는 아직 보안성을 높이기 위한 부가적인 정보일 뿐 지금의 알고리즘에 꼭 필요하지는 않습니다.
마지막으로 확정된 인증블록 대신 아직 확정되지 않은 마지막 인증블록의 정보를 포함시킬 수 있습니다.
이 후보블록의 관련성값이 인증 확정을 위한 임계값을 넘어서면 해당 인증블록의 인증이 확정된 것으로 간주할 수 있습니다.
그러나 이는 적지 않은 시간이 소요되고 아직 확정되지 않은 정보가 추가되는 후보블록들에 포함되어 서비스거부공격(denial of service attack)에 사용될 수도 있습니다.

<br/>

## 51% 공격과 지역화 ( 51% Attack and Localization )

작업증명과 마찬가지로 51% 공격(51% attack)과 지역화(localization) 문제가 발생합니다.
그러나 사용되는 소유정보와 관련성을 산출하는 방법에 따라 문제들을 최소화 시킬 수 있습니다.
다수의 노드들이 모인 풀(pool)이 운용될 수 있습니다.
이 경우엔 임의적으로 지역(zone)을 나누고 소속된 지역 정보를 관련성 산출에 포함시킬 수 있습니다.
지역화의 경우에도 위상학적 위치정보를 관련성 산출에 포함시키는 방법들을 생각해 볼 수 있습니다.
그러나 관련성에 의한 증명에서는 새로 추가된 블록이 인증을 발행하기까지는 매우 긴 시간동안 많은 협의를 거치므로 모든 네트워크 노드의 검증을 거치게 됩니다.

<br/>

## 소유정보 ( Proprietary Information )

관련성에 의한 증명은 누구나 참여 가능하도록 허가를 받지 않는 퍼블릭 블록체인(permissionless public blockchain)과 허가된 사용자만 참가할 수 있는 프라이빗 블록체인(permissioned private blockchain) 모두에 사용될 수 있습니다.
관련성 산출을 위한 소유정보가 무엇이고 어떻게 관련성을 산출하느냐에 따라 다양한 목적으로 사용될 수 있습니다.
지분증명의 지분이 사용될 수도 있습니다.
그러기 위해선 관련성 산출을 위한 지분 간에 관계 정립이 필요합니다.
작업증명과 연결하기 위해서는 블록에 포함된 작업이 특정 노드의 것이라는 소유권을 증명할 수 있어야 합니다.
소유정보의 사용으로 인해 개인의 신원이 드러날 수도 있습니다.
체인이 사용되는 목적에 맞게 영지식증명(zero Knowledge proof) 등의 방법이 함께 사용되어야 합니다.
약속된 다수의 소유정보를 함께 사용할 수도 있고 그들 간에 관계를 지어 사용할 수도 있습니다.
프라이빗 블록체인으로 운용할 경우에는 참여를 원하는 노드들에게 각각 고유한 소유정보를 미리 발행해 주는 방법을 사용할 수 있습니다.
실세계에서 이를 바로 적용해 볼 수 있는 곳이 은행권에서 발행한 공개키 인증서를 이용한 은행 거래 업무들입니다.
이 문서의 예에서도 소유키를 소유정보로 사용하였고 이는 실제 구현에서도 사용될 수 있습니다.

<br/>

## 확장성 ( Scalability )

관련성에 의한 증명은 다양한 네트워크 환경에서 사용될 수 있습니다.
이를 위해서는 각 네트워크 환경에 맞게 난이도, 관련성 상수, 관련성 효율비, 임계값, 특정 권한을 갖는 리더 후보 그룹의 비율 등의 기준값들에 대한 조정이 필요합니다.
아직은 각 기준값들에 대한 완전한 산출식들이 만들어지지 않았습니다.
지금은 어느 정도의 큰 값들을 사용하여 시뮬레이션을 통해 성능을 가늠해 볼 수 있습니다.
관련성 산출식을 기준으로 각 기준값들과의 관계가 정립이 되고 완전한 관계식들이 만들어지면 체인의 운용 상태에 따라 동적으로 조절되는 기준값들을 사용할 수 있습니다.

알고리즘을 실제 구현하는데 고려되어야 할 보안 스펙은 체인의 운용 환경과 요구되는 보안 강도에 따릅니다.
소유키를 소유정보로 사용하는 경우는 해시와 서명 생성에 각각 SHA-256과 EC-256 등을 사용할 수 있습니다.
이 때 가질 수 있는 최고 난이도 값은 256으로 이 값에 맞추어 적당한 난이도 값이 채택 되어야 합니다.
이중 체인의 사용은 단일 체인을 사용하는 합의 알고리즘보다 하나의 인증 발행을 위해 더 많은 공간을 필요로 합니다.
아직은 확장성(scalability)과 관련하여 충분히 고려가 되지는 않았습니다.
난이도에 의해 고정되는 앞 상수 비트들을 기준으로 체인을 트리 형태로 분리하는 방법들을 생각해 볼 수 있습니다.
고정되는 앞 상수 비트들이 연속되는 0이 되도록 첫 블록을 발행하여 사이드체인(sidechain)으로의 가능성을 고려해 볼 수 있습니다.

<br/>

###### ※ 이 알고리즘은 코인(coin) 발행을 목적으로 고안되지는 않았습니다.<br/><br/>알고리즘을 구현하고 검증하는데 필요한 자금을 확보하기 위해 코인 발행이 필요할 수도 있습니다.<br/><br/>코인이 발행되더라도 사익이 아니라 알고리즘의 개념증명(proof of concept)이 목적이 되어야 합니다.<br/><br/>보상 체계는 필요하고 이는 여타 스마트계약(smart contract) 관련 알고리즘들의 방법을 따릅니다.<br/><br/>이 알고리즘이 블록체인과 관련된 많은 난제들을 해결하여 블록체인 기술이 활성화되길 바랍니다.
