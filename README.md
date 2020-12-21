# 부트페이 소개 

부트페이는 이니시스, 네이버페이, 카카오페이 등 모든 PG와 결제수단을 간편하게 결제연동 할 수 있습니다. PG나 결제수단 변경시 parameter 만 바꾸면 되며, 더욱 개발하기 쉽도록 인터페이스를 제공합니다.


## next-js-bootpay-example

부트페이의 javascript 파일을 cdn에서 불러와 next.js 프로젝트에서 사용할 수 있습니다. next.js 어떻게 bootpay.js 를 사용하는지에 대해 다룹니다. 


## pages/_document.js 수정

만약 해당 파일이 없다면 생성해주세요. _document.js는 SPA에서 시작점이 되는 index.html이라고 생각하면 되는데, Custom Document를 만들때만 작성이 필요하며, 생략된 경우 Next.js가 기본 값을 사용합니다.
```javascript
import Document, { Head, Main, NextScript } from 'next/document';

export default class RootDocument extends Document {
    render() {
        return (
            <html>
                <Head>
                    <meta charSet="utf-8" />
                    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1, user-scalable=no" />
                    <meta name="description" content="Dev.log"/>
                    <meta name="keywords" content="blog,react,antd,webpack,css,javascript" />
                    <link rel="manifest" href="/static/manifest.json" />
                    <link rel="shortcut icon" href="/static/favicon.ico" /> 
                    <script src="https://cdn.bootpay.co.kr/js/bootpay-3.3.1.min.js" type="application/javascript"></script>
                </Head>
                <body>
                    <Main />
                    <NextScript />
                </body>
            </html>
        );
    }
}
```

기존 파일이 있었다면 head 태그안에 script 만 삽입하시면 됩니다.
```javascript
...
<script src="https://cdn.bootpay.co.kr/js/bootpay-3.3.1.min.js" type="application/javascript"></script>
...
```



## 사용하기 


```javascript 
import styles from './index.module.css'

const Index = () => (
  <div className={styles.container}>
    <div className="box">
      <button onClick={onClickRequest}>부트페이 결제호출</button>
    </div>
  </div>
);

// componentdi

function onClickRequest() {
  window.BootPay.request({
    price: '1000', //실제 결제되는 가격
    application_id: "5b8f6a4d396fa665fdc2b5e7",
    name: '블링블링 마스카라', //결제창에서 보여질 이름
    pg: 'kcp',
    method: 'card', //결제수단, 입력하지 않으면 결제수단 선택부터 화면이 시작합니다.
    show_agree_window: 0, // 부트페이 정보 동의 창 보이기 여부
    items: [
      {
        item_name: '나는 아이템', //상품명
        qty: 1, //수량
        unique: '123', //해당 상품을 구분짓는 primary key
        price: 1000, //상품 단가
        cat1: 'TOP', // 대표 상품의 카테고리 상, 50글자 이내
        cat2: '티셔츠', // 대표 상품의 카테고리 중, 50글자 이내
        cat3: '라운드 티', // 대표상품의 카테고리 하, 50글자 이내
      }
    ],
    user_info: {
      username: '사용자 이름',
      email: '사용자 이메일',
      addr: '사용자 주소',
      phone: '010-1234-4567'
    },
    order_id: '고유order_id_1234', //고유 주문번호로, 생성하신 값을 보내주셔야 합니다.
    params: { callback1: '그대로 콜백받을 변수 1', callback2: '그대로 콜백받을 변수 2', customvar1234: '변수명도 마음대로' },
    account_expire_at: '2020-10-25', // 가상계좌 입금기간 제한 ( yyyy-mm-dd 포멧으로 입력해주세요. 가상계좌만 적용됩니다. )
    extra: {
      start_at: '2019-05-10', // 정기 결제 시작일 - 시작일을 지정하지 않으면 그 날 당일로부터 결제가 가능한 Billing key 지급
      end_at: '2022-05-10', // 정기결제 만료일 -  기간 없음 - 무제한
      vbank_result: 1, // 가상계좌 사용시 사용, 가상계좌 결과창을 볼지(1), 말지(0), 미설정시 봄(1)
      quota: '0,2,3', // 결제금액이 5만원 이상시 할부개월 허용범위를 설정할 수 있음, [0(일시불), 2개월, 3개월] 허용, 미설정시 12개월까지 허용,
      theme: 'purple', // [ red, purple(기본), custom ]
      custom_background: '#00a086', // [ theme가 custom 일 때 background 색상 지정 가능 ]
      custom_font_color: '#ffffff' // [ theme가 custom 일 때 font color 색상 지정 가능 ]
    }
  }).error(function (data) {
    //결제 진행시 에러가 발생하면 수행됩니다.
    console.log(data);
  }).cancel(function (data) {
    //결제가 취소되면 수행됩니다.
    console.log(data);
  }).ready(function (data) {
    // 가상계좌 입금 계좌번호가 발급되면 호출되는 함수입니다.
    console.log(data);
  }).confirm(function (data) {
    //결제가 실행되기 전에 수행되며, 주로 재고를 확인하는 로직이 들어갑니다.
    //주의 - 카드 수기결제일 경우 이 부분이 실행되지 않습니다.
    console.log(data);
    var enable = true; // 재고 수량 관리 로직 혹은 다른 처리
    if (enable) {
      BootPay.transactionConfirm(data); // 조건이 맞으면 승인 처리를 한다.
    } else {
      BootPay.removePaymentWindow(); // 조건이 맞지 않으면 결제 창을 닫고 결제를 승인하지 않는다.
    }
  }).close(function (data) {
    // 결제창이 닫힐때 수행됩니다. (성공,실패,취소에 상관없이 모두 수행됨)
    console.log(data);
  }).done(function (data) {
    //결제가 정상적으로 완료되면 수행됩니다
    //비즈니스 로직을 수행하기 전에 결제 유효성 검증을 하시길 추천합니다.
    console.log(data);
  });
}

export default Index;
```
