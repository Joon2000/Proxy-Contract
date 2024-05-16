# Proxy Contract

##### 프록시 컨트랙트가 필요한 주된 이유는 Ethereum 스마트 컨트랙트의 기본적인 제한 사항, 즉 한번 배포된 후 코드를 변경할 수 없는 불변성을 극복하기 위해서입니다. 이러한 불변성은 보안과 신뢰성을 강화하지만, 소프트웨어 개발에서 일반적으로 요구되는 유연성과 확장성을 제한합니다. 개발자들은 proxy contract를 통해 기존의 스마트 컨트랙트를 업그레이드 할 수 있으며, 버그 수정이나 기능 추가 같은 변경사항을 적용할 수 있습니다. 이런 유연성은 블록체인 애플리케이션의 개발과 유지보수를 보다 쉽고 효과적으로 만들어 줍니다.<br>

<img src="https://github.com/Joon2000/Proxy-Contract/blob/main/images/ProxyContract%20structure.png" width="80%" height="60%" alt="Proxy structure"></img><br><br>

##### 기존의 컨트랙트의 버그를 고치거나 새로운 로직을 추가하기 위해서 스마트 컨트랙트를 새로 작성하여 배포하면 기존의 데이터가 모두 날라갈 뿐만 아니라 새로운 컨트랙트 주소로 배포됩니다. 프록시 컨트랙트는 data를 저장하는 contract와 logic을 수행하는 contract를 분리하면서 이 문재를 해결합니다. 
##### data와 address를 보존하는 contract를 proxy contract라고 하며 logic을 담고 있는 contract를 implementation contract라고 합니다. Proxy contract는 데이터 보존을 위해서 delegate call을 통해서 implementation contract의 함수를 call 합니다.


## 실습
##### 이번 실습에서는 proxy contract를 직접 코딩하기보다는 openzeppelin의 proxy contract를 사용해보도록 하겠습니다.

### 중요! 이번 실습은 sepolia testnet에 배포합니다

### 1. Remix에 빈 workspace를 만든 뒤 다음 두 contract(implementation contract와 Proxy contract)를 contracts 폴더 안에 만듭니다.<br>

> #### Implementation Contract
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

> #### Proxy Contract
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
> Remix 폴더 구조 (contracts 폴더만 있으면 됩니다)<br>
<img src="https://github.com/Joon2000/Proxy-Contract/blob/main/images/Proxy%20contract%20folder.png" width="30%" height="60%" alt="Proxy structure"></img><br><br>

### 2. 컴파일 후 배포
> 두 contract를 모두 컴파일 한 후 sepolia에 배포합니다. 이때 implementation contract 먼저 deploy한 후 이 address를 사용하여 proxy contract를 deploy합니다.

### 3. 배포된 컨트랙트 etherscan에서 verify하기 
<img src="https://github.com/Joon2000/Proxy-Contract/blob/main/images/proxycontract%20button.png" width="30%" height="60%" alt="Proxy structure"></img><br><br>
> Rexmix의 proxt contraxt의 버튼을 보면 위의 사진과 같이 implementation contract의 함수의 버튼이 보이지 않는다. Remix가 delegate call을 인식 못하는 건데 abi를 통해서도 확인할 수 있습니다. (이번 실습은 implementation contract의 setText 함수를 proxy contract에서 호출해볼려고 합니다.)<br>
```javascript
[
	{
		"inputs": [
			{
				"internalType": "address",
				"name": "_implementation",
				"type": "address"
			}
		],
		"stateMutability": "nonpayable",
		"type": "constructor"
	},
	{
		"inputs": [],
		"name": "admin",
		"outputs": [
			{
				"internalType": "address",
				"name": "",
				"type": "address"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "proxy",
		"outputs": [
			{
				"internalType": "address",
				"name": "",
				"type": "address"
			}
		],
		"stateMutability": "view",
		"type": "function"
	}
]
```
> 위의 abi를 보면 implementation contract의 setText함수가 보이지 않는 것을 확인할 수 있습니다.<br> 
> #####따라서 이번 실습에서는 remix가 아닌 etherscan에서 함수를 동작시켜보겠습니다.<br>

> 먼저 remix의 왼쪽 아래 plugin manager를 클릭 후 CONTRACT VERIFICATION - ETHERSCAN을 activate 시켜줍니다.<br>
<img src="https://github.com/Joon2000/Proxy-Contract/blob/main/images/plugin%20Etherscan.png" width="30%" height="60%" alt="Proxy structure"></img><br><br
> etherscan 로근인 후 api key를 발급받아 입력해줍니다.<br>
> implementation contract 먼저 verify 합니다.<br>
>> contract name에 HelloWorldV1을 선택하고 address를 입력 후 verify 해줍니다.<br>
<img src="https://github.com/Joon2000/Proxy-Contract/blob/main/images/HelloWorldV1%20verify.png" width="30%" height="60%" alt="Proxy structure"></img><br><br>
> 다음으로 proxt contract를 verify 해줍니다.<br>
>> 이때 It's a proxy contract address를 선택해주고 implementation contract 주소를 기입합니다<br>
>> Constructor Arguments도 채워줍니다.<br>
<img src="https://github.com/Joon2000/Proxy-Contract/blob/main/images/proxy%20contract%20verify.png" width="30%" height="60%" alt="Proxy structure"></img><br><br>

### etherscan에서 함수 동작하기
> etherscan sepolia(https://sepolia.etherscan.io/)에서 proxy contract address로 scan합니다.
> 


이번 tutorial은 solidity 공식문서가 아닌 https://jamesbachini.com/proxy-contracts-tutorial/로부터 MIT license 코드를 사용했습니다.
