# Calculate Smartcontract code 

#### Caution

- 소수점은 다섯자리, 남는 차액은 엑스피어에서 가져간다. 
- 약관에 무조건 표기 
- 프로젝트 지갑을 따로 프로젝트별로 생성, 토큰 락, NFT 모두 이곳에 저장해서 관리 
- 정산 받을 사람들의 수수료를 나눴을 때 정확히 떨어지지 않는 에러가 발생할 수 있음
  - 소수점 제한이 화면에 표시되는 것과 kXDT의 소수점이 다르기 때문 
  - ~~제한조건을 만들어줘야 함~~
  - 어드민 프로그램에서 계산을 해서 지갑주소 : 보낼 토큰 양으로 정해져서 보내주는 것이 좋음
  - 컨트랙트 상에서는 간단하게 누구에게 얼마가 가는지만 전달하여 Token Transfer만 전송하게 해주는 것이 
  좀 더 안정적일 것으로 보임
- 함수를 실행하는 실행자에 대한 조건이 필요 (owner, 또는 어드민 계정)

#### Process

![Calculate Process](./img/calculate_process.jpg)

#### Action 

- 어드민페이지의 정산 페이지에서 옥션의 결과를 전달 받는다.(구매자, 최고 낙찰가, tokenId)
~~- 옥션 판매에 대한 수수료(xpeare) 를 입력 받는다.~~
- 수수료를 뺀 나머지 금액을 보여준다. 
- 정산 받을 주소를 입력 받는다. 
- 정산받을 각 주소에 대한 금액을 입력 받는다. 
- (주소 : 토큰 수수료) 리스트를 만든다. 
- Creator 정산 완료 버튼을 클릭
  - 각 정산 받을 주소에 맞는 토큰 수량을 전송해준다.
- NFT 낙찰자에게 전송 버튼 클릭 
  - 최종 낙찰자에게 해당하는 NFT를 전송해준다.

#### Struct 

- **Creator Calculate**
  - 최종 낙찰 금액 (정산 받을 금액들의 합산 체크를 위해)
  - 거래 수수료(xpeare가 받는 금액)
  - 정산 받을 사람 리스트 (정산받을 지갑주소와 정산 받을 토큰 수량 매핑, xpeare wallet 지갑 주소 포함)

- **NFT Calculate**
  - 낙찰자 (NFT 받을 사람)
  - NFT tokenId
  - 최종 낙찰 금액 (정산 받을 금액들의 합산 체크를 위해)

#### Event

- 정산 금액 전송 
- NFT 전송 

#### Function

- creator에게 정산 금액 전송
- 최종 낙찰자에게 NFT 전송 

#### Modifier 

- whenPaused
  - 컨트랙트가 정지되었는지 체크
- whenNotPaused
  - 컨트랙트가 정지가 안되었는지 체크
- onlyOwner
  - 컨트랙트 소유자인지 체크 

```
 struct CreatorCalculate {
  endingPrice; //최종 낙찰 금액 
  tradeFee; //거래 수수료 
  creatorList; //정산 리스트(지갑주소 : 금액)
 }
 
 struct NFTCalculate {
  nftTokenId; // 옥션으로 낙찰된 NFT tokenId
  auctionWonAddress; //최종 낙찰자 주소 
 }
 
 event CreatorCalculateCreated(CreatorCalculate); //Creator 정산 생성 
 event NFTCalculateCreated(NFTCalculate); // NFT 정산 생성
 event CreatorCalcuate(CreatorCalculate); // 정산 금액 전송  
 event NFTCalculate(NFTCalculate); // NFT 낙찰자에게 전송

 function createCreatorCalculate(endingPrice, tradeFee, creatorList); //Creator 정산 생성 
 function createNFTCalculate(nftTokenId, auctionWonAddress); // NFT 정산 생성
 function widthdrawCreator(CreatorCalcuate); // 정산 금액 전송
 function widthdrawNFT(NFTCalculate); // NFT 낙찰자에게 전송
 
 function _widthdrawCreator(_creator, _amount);
 function _widthdrawNFT(_receiver, _nftTokenId);
```


