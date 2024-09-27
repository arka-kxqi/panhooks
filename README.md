# PanHooks: Cross-Chain Perpetuals Market Enhancing PancakeSwap V4 with OFTs


#### **Introduction**

PanHooks is a cross-chain perpetual futures market built on top of PancakeSwap V4 Pools. It uses ERC20 tokens wrapped in LayerZero's Omnichain Fungible Tokens (OFTs) to facilitate seamless cross-chain asset movement and collateral management. The platform operates permissionlessly and without the need for oracles, allowing users to trade futures and options on any PancakeSwap V4 pool.

#### **Project Description**

PanHooks is a cross-chain perpetual futures market that interacts with PancakeSwap V4 pools using ERC20 tokens wrapped as OFTs. This provides users with the ability to trade and manage collateral across different blockchains. The current implementation supports V4 pools with USDC as one of their assets, and our novel OFT design allows collateral to be deposited from any chain into the protocol.

By extending LayerZero's OFT functionality, PanHooks enables any ERC20 token to be wrapped into an OFT and deposited as collateral via our custom "sendAsCollateral" function. This enhances the cross-chain capabilities of PancakeSwap V4 pools, making liquidity more accessible and versatile.

#### **How It’s Made**

PanHooks relies heavily on PancakeSwap V4 pools and LayerZero's OFTs. By modifying the default OFT functionality, we allow any ERC20 token to be wrapped into an OFT and sent as cross-chain collateral to the PanHooks market.

The key innovation lies in our token "OFTWrappedERC20," which exposes a custom function "sendAsCollateral." This function allows wrapped ERC20 tokens to be deposited cross-chain, improving liquidity management across networks. Our implementation is inspired by Hook Finance and expands on the idea by adding significant cross-chain functionality.

For further technical details and deployed contract addresses, refer to the [GitHub monorepo](https://github.com/panhooks).

#### **Key Features**

- **Cross-Chain Perpetuals:** Enables trading of perpetual futures across multiple chains with wrapped ERC20 tokens.
- **Custom OFT Functionality:** Modified OFTs allow tokens to be sent as collateral cross-chain, increasing liquidity.
- **Permissionless and Oracle-Free:** PanHooks allows anyone to create and trade futures on PancakeSwap V4 pools without needing price oracles.
- **PancakeSwap V4 Pool Integration:** Seamlessly integrates with any V4 pool to provide enhanced trading options.

#### **How PanHooks Enhances PancakeSwap V4 Ecosystem**

1. **Cross-Chain Liquidity Expansion:** Allows users to utilize liquidity from multiple chains via OFTs, enhancing PancakeSwap V4 pools.
2. **Custom Cross-Chain Collateral:** Introduces new functionality with "sendAsCollateral," enabling a novel way to manage cross-chain collateral.
3. **Decentralized Futures Market:** Empowers users to create and trade futures markets on any PancakeSwap V4 pool, increasing overall liquidity and market participation.
4. **Oracle-Free Mechanism:** By operating without external price feeds, PanHooks increases decentralization and reduces reliance on external sources.


### **Usage**

Below is a sample code that demonstrates how the **PanHooks** contract integrates with LayerZero's OFT functionality to facilitate cross-chain collateral deposits using PancakeSwap V4 pools. This code highlights the key `sendAsCollateral` function and how it interacts with the PancakeSwap pools and Captain Hook's custom functionality.

```solidity
import {SendParam, MessagingReceipt, OFTReceipt} from "@layerzerolabs/oft/interfaces/IOFT.sol";
import {MessagingFee, Origin} from "@layerzerolabs/oft/interfaces/ILayerZeroEndpointV2.sol";
import {OFTComposeMsgCodec} from "@layerzerolabs/oft/libs/OFTComposeMsgCodec.sol";
import {OFTMsgCodec} from "@layerzerolabs/oft/libs/OFTMsgCodec.sol";
import {OFTWrappedERC20} from "./OFTWrappedERC20.sol";
import {PoolKey} from "./PancakeV4Structs.sol";
import {ICaptainHook} from "./ICaptainHook.sol";

contract CaptainHookOFT is OFTWrappedERC20 {
    ICaptainHook public captainHook;

    using OFTMsgCodec for bytes;
    using OFTMsgCodec for bytes32;

    constructor(
        string memory _name, 
        string memory _symbol, 
        address _lzEndpoint, 
        address _delegate, 
        address _baseToken
    ) OFTWrappedERC20(_name, _symbol, _lzEndpoint, _delegate, _baseToken) {}

    function setCaptainHook(address _captainHook) external {
        require(address(captainHook) == address(0), "Captain Hook already set");
        captainHook = ICaptainHook(_captainHook);
    }

    // Sends ERC20 tokens wrapped as OFT cross-chain and deposits them as collateral into a PanHooks pool
    function sendAsCollateral(
        SendParam calldata _sendParam,
        MessagingFee calldata _fee,
        address _refundAddress,
        PoolKey calldata key
    ) external payable returns (MessagingReceipt memory msgReceipt, OFTReceipt memory oftReceipt) {
        (uint256 amountSentLD, uint256 amountReceivedLD) = _debit(_sendParam.amountLD, _sendParam.minAmountLD, _sendParam.dstEid);
        (bytes memory message, bytes memory options) = _buildMsgAndOptions(_sendParam, amountReceivedLD);
        bytes memory keyWithMessage = abi.encode(key, message);
        
        msgReceipt = _lzSend(_sendParam.dstEid, keyWithMessage, options, _fee, _refundAddress);
        oftReceipt = OFTReceipt(amountSentLD, amountReceivedLD);
        
        emit OFTSent(msgReceipt.guid, _sendParam.dstEid, msg.sender, amountSentLD, amountReceivedLD);
    }
}
```

#### **Key Functions Explained:**

- `sendAsCollateral`: This function allows the ERC20 tokens wrapped as OFTs to be sent cross-chain and deposited as collateral into PancakeSwap V4 pools managed by the Captain Hook contract. It uses LayerZero’s messaging system to ensure safe and accurate cross-chain transfers.
  
- **Helper Functions**:
  - `sendTo`: Extracts the recipient’s address from the message.
  - `amountSD`: Retrieves the amount being sent from the message.
  - `isComposed`: Checks if the message includes composed operations to be executed after the collateral send.

#### **Example Use Case**:
A user wants to trade perpetual futures on PanHooks using ERC20 tokens from a different blockchain. They can wrap these tokens into OFTs and send them cross-chain as collateral using the `sendAsCollateral` function, ensuring seamless integration with PancakeSwap V4 pools.


#### **Contracts Overview**

- **Optimism Sepolia:**
  - (Fake) USDC: `0xcE1Fd4B4CD044dfc4b488633A4BEBd9AB61E116a`
  - Amazing Token (AT): `0xFc5cea8407Be982eCfA64041cFfe85A69f32c0B2`
  - OFT Wrapped USDC: `0xAd6cD4472D30911e660BD19024fD36Ef5f126e89`
  - OFT Wrapped AT: `0x105fd7E5A68A5ab2545e49Da021F94cb73D042b8`
  
- **Base Sepolia:**
  - PanHooks: `0xD9f64CA93CDB3c5F7DE29E975b0dBE9e2446F6Aa`
  - Vault: `0x5099c42484665CD0fd12Fc2EEBFeF2dC20948F1b`
  - CLPoolManager: `0x6ff773dAD3B7f16c528A2d55fA3Fe460e78A0Def`
  - wUSDC: `0xAdFCbF60d5C5b8739B458602Aa64f7313c0D9FF1`
  - wAT: `0xBD913298f2E2170a87B75Dc851fa9Fa5394dF48A`

#### **Installation & Usage**

1. **Clone the Repository**
   ```bash
   git clone https://github.com/panhooks
   cd panhooks
   ```

2. **Set Up Contracts**  
   Deploy or interact with the provided addresses on Optimism Sepolia or Base Sepolia test networks.

3. **Integrate with PancakeSwap V4**  
   Integrate PanHooks’ custom hooks to create cross-chain futures markets on PancakeSwap V4 pools. Documentation on integration and customization of the hooks can be found in the monorepo.

#### **Conclusion**

PanHooks enhances the PancakeSwap V4 ecosystem by offering cross-chain perpetual futures trading, novel collateral management through modified OFTs, and permissionless market creation. Its oracle-free and decentralized structure provides users with a more flexible and robust trading platform, expanding the possibilities of decentralized finance.