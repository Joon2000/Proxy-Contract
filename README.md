# Proxy Contract

##### 프록시 컨트랙트가 필요한 주된 이유는 Ethereum 스마트 컨트랙트의 기본적인 제한 사항, 즉 한번 배포된 후 코드를 변경할 수 없는 불변성을 극복하기 위해서입니다. 이러한 불변성은 보안과 신뢰성을 강화하지만, 소프트웨어 개발에서 일반적으로 요구되는 유연성과 확장성을 제한합니다. 개발자들은 proxy contract를 통해 기존의 스마트 컨트랙트를 업그레이드 할 수 있으며, 버그 수정이나 기능 추가 같은 변경사항을 적용할 수 있습니다. 이런 유연성은 블록체인 애플리케이션의 개발과 유지보수를 보다 쉽고 효과적으로 만들어 줍니다.



##### 기존의 컨트랙트의 버그를 고치거나 새로운 로직을 추가하기 위해서 스마트 컨트랙트를 새로 작성하여 배포하면 기존의 데이터가 모두 날라갈 뿐만 아니라 새로운 컨트랙트 주소로 배포됩니다. 프록시 컨트랙트는 data를 저장하는 contract와 logic을 수행하는 contract를 분리하면서 이 문재를 해결합니다. 
##### data와 address를 보존하는 contract를 proxy contract라고 하며 logic을 담고 있는 contract를 implementation contract라고 합니다. Proxy contract는 데이터 보존을 위해서 delegate call을 통해서 implementation contract의 함수를 call 합니다.


### 실습




```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract HelloWorldV1 {
    string public text = "hello world";

    function setText(string memory _text) public {
        text = _text;
    }
}
```

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/proxy/transparent/ProxyAdmin.sol";
import "@openzeppelin/contracts/proxy/transparent/TransparentUpgradeableProxy.sol";

contract ProxyDeployer {
	address public admin;
	address public proxy;
	constructor(address _implementation) {
		ProxyAdmin adminInstance = new ProxyAdmin(msg.sender);
		admin = address(adminInstance);
		TransparentUpgradeableProxy proxyInstance = new TransparentUpgradeableProxy(_implementation, admin, "");
		proxy = address(proxyInstance);
	}
}
```

이번 tutorial은 solidity 공식문서가 아닌 https://jamesbachini.com/proxy-contracts-tutorial/로부터 MIT license 코드를 사용했습니다.
