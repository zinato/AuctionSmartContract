# Auction Smartcontract Code 

#### Action
1. 어드민에서 옥션 실행 -> 컨트랙트에서 옥션 생성
2. 어드민에서 옥션 취소 ->  mapping된 옥션 정보 삭제
3. 어드민에서 종료된 옥션 리스트 보기
   1. 어드민에서 옥션 종료 -> 컨트랙트에서 옥션 종료, mapping된 옥션 정보 삭제 및 옥션 결과 리턴


#### Struct 

- **Auction**
  - 옥션 기간 (시간 타입 정해야 함)
    - 옥션 시작 시간 
    - 옥션 종료 시간
  - 판매가
  - 판매자
  - NFT 토큰 Id

[comment]: <> (    - 옥션 시작가)

[comment]: <> (    - 옥션 종료가)
 

- **Bid**
  - 입찰자 주소 
  - 입찰가 
  - 이전에 입찰한 사람인지 
  - 참여하는 옥션의 tokenId
    - 옥션의 mapping 정보가 tokenId 로 mapping 되어야 함

#### Event (Log)

- 옥션 생성
- 옥션 취소 
- 옥션 종료
- 최고입찰가 변경 
- 입찰가 반환 

#### Mapping

- tokenId와 매핑된 Auction 정보
- tokenId와 address(구매자)와 매핑된 최고입찰가 정보  
  - 이게 리스트로 만들어질 수 있고 효율적이면 입찰내역 전체 리스트는 만들지 않아도 됨
- tokenId와 address(구매자)와 매핑된 입찰내역 전체 리스트

#### Function

- 옥션 생성 
  - 옥션 매핑 정보에 저장되고 seller의 nft가 CA로 전송됨
  - AuctionCreated 이벤트 호출  

- 옥션 취소 
  - 해당 옥션이 mapping된 정보에서 삭제되고 seller에게 nft가 다시 돌아감
  - 입찰자가 있으면 입찰자에게 입찰가를 다시 돌려줌
  - AuctionCanceled 이벤트 호출 

- 옥션 현재 정보 반환 
  - tokenId로 매핑된 옥션 정보에서 해당 옥션이 현재 진행중인지 
  - seller와 현재가가 얼마인지 리턴해주는 함수 

- 옥션 결과 제공 (또는 옥션 종료 mapping을 만들어 mapping 정보에서 제공)
  - 어드민에서 옥션 종료가 되기전에 검수를 위해 옥션에 현재 최종 입찰가와 판매자를 확인할 수 있는 함수
  - 여기서 모두 확인이 되어 어드민에서 옥션 종료 버튼을 눌러 옥션을 종료 시킴 

- 옥션 종료
  - 옥션기간이 끝나면 어드민에서 특정 버튼을 통해 옥션을 종료
  - 옥션 mapping 정보에서 종료된 옥션 정보 제거 
  - mapping 정보에서 Bid 관련 정보 제거 
  - 최종 입찰가, 구매자 정보를 리턴
  - AuctionSuccess Event 호출 

- 입찰하기 
  - 옥션 기간 체크
  - 해당 옥션에 입찰한 내역이 있는지 체크
  - 지갑 잔액 체크 
  - 입찰가를 CA로 전송 
  - 최고 입찰가로 등록
  - tokenIdToAuction 옥션 정보 업데이트
  - 입찰 내역 mapping 정보에 등록
  - 이전 입찰자에게 이전 입찰가를 돌려줌
  - BidChanged 이벤트 호출 
  - BidRefund 이벤트 호출
    - 이전 최고 입찰가 정보

- 현재 입찰가를 반환 
  - tokenIdToAuction[tokenId] 매핑된 정보에서 Auction 정보를 가져와
    - Auction이 진행중인지 체크 
    - 해당 옥션의 tokenId로 입찰가 리스트에서 
    - 

- 입찰 내역 전체 리스트 반환

#### Modifier

- whenPaused
  - 컨트랙트가 정지되었는지 체크
- whenNotPaused
  - 컨트랙트가 정지가 안되었는지 체크
- onlyOwner
  - 컨트랙트 소유자인지 체크 

```
  struct Auction {
    startingPrice; //옥션 시작가
    endingPrice; //옥션 종료가
    startTime; // 옥션 시작 시간
    endTime; // 옥션 종료 시간
    seller; //현재 NFT 소유자
    tokenId; //nft TokenId
  }

  struct Bid {
    hasBid, //입찰 참여내역
    bidderId, // 입찰 참여자 주소
    price //입찰 가격
  }

  event AuctionCreated; //옥션 생성
  event AuctionCanceled; // 옥션 취소
  event AuctionSuccess; // 옥션 성공 (종료) 
  event BidChanged; // 최고가 입찰 변경
  event BidRefunded; //이전 최고 입찰자에게 입찰가를 돌려준후 찍는 로그 
  
  //tokenId로 해당하는 Auction 매핑 
  mapping (uint256 => Auction) tokenIdToAuction;
  //tokenId와 address(구매자)와 매핑된 최고입찰가 정보 => Auction 에 Price로도 가져올 수 있음
  mapping(uint256 => Bid))) highestBiddingPriceByTokenId;
  //tokenId와 address(구매자)와 매핑된 입찰내역 전체 리스트
  mapping(uint256 => mapping(uint256 => Bid))) bidsListByTokenId;
  
  function createAuction //옥션 생성
  function cancelAuction //옥션 취소 
  function getAuction // 옥션 정보 리턴 
  function getCurrentPrice //현재가 리턴 
  function closeAuction //옥션 종료 
  function placeBid // 입찰하기
  function getBidList; //입찰 전체 내역 리스트 
     
  function _addAuction(_tokenId, auction);
  funtcion _removeAuction(_tokenId);
  function __isOnAuction(Auction);
  function _getAuction(Auction);
  function _getCurrentPrice(Auction);
  function _closeAuction;
  function _placeBid;  
  funciton _getBidList;
   
```
