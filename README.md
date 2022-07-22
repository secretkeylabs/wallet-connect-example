Wallet Connect for Stacks 
=========================

Helper library and example code ilustrating how to implement Wallet Connect 2 with Stacks based wallet ([Xverse](https://www.xverse.app/)).

For instructions on **how to develop THIS package**, see: [Local development](#local-development).

For instructions on **how to integrate Wallet Connect with Stacks wallet into your app**, see: [Integration tutorial](#integration-tutorial).

# Apps

1. [Wallet](/wallet/README.md) - react app that simulates a wallet

    Original repo: [https://github.com/WalletConnect/web-examples/tree/main/wallets/react-wallet-v2](https://github.com/WalletConnect/web-examples/tree/main/wallets/react-wallet-v2)

2. [Web App](/webapp/README.md) - react app that we implement the Wallet Connect to.

    Original repo: [https://github.com/WalletConnect/web-examples/tree/main/dapps/react-dapp-v2](https://github.com/WalletConnect/web-examples/tree/main/dapps/react-dapp-v2)

3. [Stacks Wallet Connect](/stacks-wallet-connect/README.md) - helper functions and constants

# Local development

## Setup

First, clone the repo (obviously)

### Generate an OpenConnect Project ID
You can generate your own ProjectId at https://cloud.walletconnect.com

### Install **stacks-wallet-connect** dependencies and make it linkable
```bash
cd stacks-wallet-connect
yarn
yarn link
cd ..
```
### Set up and start **wallet** 
1. Open a new terminal
2. Build and link the **stacks-wallet-connect** module
```bash
cd wallet
yarn
yarn build  # TODO Fix the compile error
yarn link @web3devs/stacks-wallet-connect
```
2. Create a local environment
```bash
cp .env.local.example .env.local
```
3. Add your WalletConnect Project ID to `NEXT_PUBLIC_PROJECT_ID`.
4. Start the wallet app
```bash
yarn start
```

#### Compile errors
The compile in step 2 fails with **many** errors. They seem to be the kinds of failures caused by a missing dependancy. Unfortunately, I don't know what the dependancy is.
```text
Failed to compile.

./src/pages/_app.tsx
1:20  Error: "@/components/Layout" is not found.  node/no-missing-import
2:19  Error: "@/components/Modal" is not found.  node/no-missing-import
3:31  Error: "@/hooks/useInitialization" is not found.  node/no-missing-import
4:43  Error: "@/hooks/useWalletConnectEventsManager" is not found.  node/no-missing-import
...
```

### Set up and start the ***webapp***
1. Open a new terminal
2. Build and link the **stacks-wallet-connect** module
```bash
cd webapp
yarn
yarn build # TODO Fix the compile error
yarn link @web3devs/stacks-wallet-connect
```
2. Create a local environment
```bash
cp .env.local.example .env.local
```
3. Add your WalletConnect Project ID to `NEXT_PUBLIC_PROJECT_ID`.
4. Start the wallet app
```bash
yarn start
```

#### Compile Errors
There compile errors here are different, but also appear to be cause by a missing dependancy:
```text
Error: error:0308010C:digital envelope routines::unsupported
    at new Hash (node:internal/crypto/hash:67:19)
    at Object.createHash (node:crypto:133:10)
    at module.exports (/Users/ken/Projects/wallet-connect-example/webapp/node_modules/webpack/lib/util/createHash.js:135:53)
    at NormalModule._initBuildHash (/Users/ken/Projects/wallet-connect-example/webapp/node_modules/webpack/lib/NormalModule.js:417:16)
    at handleParseError (/Users/ken/Projects/wallet-connect-example/webapp/node_modules/webpack/lib/NormalModule.js:471:10)
    at /Users/ken/Projects/wallet-connect-example/webapp/node_modules/webpack/lib/NormalModule.js:503:5
    at /Users/ken/Projects/wallet-connect-example/webapp/node_modules/webpack/lib/NormalModule.js:358:12
    at /Users/ken/Projects/wallet-connect-example/webapp/node_modules/loader-runner/lib/LoaderRunner.js:373:3
    at iterateNormalLoaders (/Users/ken/Projects/wallet-connect-example/webapp/node_modules/loader-runner/lib/LoaderRunner.js:214:10)
    at iterateNormalLoaders (/Users/ken/Projects/wallet-connect-example/webapp/node_modules/loader-runner/lib/LoaderRunner.js:221:10)
    at /Users/ken/Projects/wallet-connect-example/webapp/node_modules/loader-runner/lib/LoaderRunner.js:236:3
    at runSyncOrAsync (/Users/ken/Projects/wallet-connect-example/webapp/node_modules/loader-runner/lib/LoaderRunner.js:130:11)
    at iterateNormalLoaders (/Users/ken/Projects/wallet-connect-example/webapp/node_modules/loader-runner/lib/LoaderRunner.js:232:2)
    at Array.<anonymous> (/Users/ken/Projects/wallet-connect-example/webapp/node_modules/loader-runner/lib/LoaderRunner.js:205:4)
    at Storage.finished (/Users/ken/Projects/wallet-connect-example/webapp/node_modules/enhanced-resolve/lib/CachedInputFileSystem.js:55:16)
    at /Users/ken/Projects/wallet-connect-example/webapp/node_modules/enhanced-resolve/lib/CachedInputFileSystem.js:91:9
/Users/ken/Projects/wallet-connect-example/webapp/node_modules/react-scripts/scripts/build.js:19
  throw err;
  ^
```

# Integration tutorial

For this tutorial we'll use two existing apps provided by Wallet Connect:

- [Wallet](https://github.com/WalletConnect/web-examples/tree/main/wallets/react-wallet-v2)
- [React Web App](https://github.com/WalletConnect/web-examples/tree/main/dapps/react-dapp-v2)

Both apps are **NOT** integrated with Stacks, so we need to build everything ourselves.

We'll cover the **WebApp** part first and then move on to how to implement Wallet Connect on **Wallet's** side.

## Create your local development environment

        mkdir project
        cd project
        git clone git@github.com:WalletConnect/web-examples.git
        cp -r web-examples/dapps/react-dapp-v2 webapp
        cp -r web-examples/wallets/react-wallet-v2 wallet
        rm -rf web-examples

    At this point you should have **wallet** and **webapp** in your project directory

## Wallet

First you need Stacks wallet that implements Wallet Connect. If you have one - go to [WebApp](#webapp) section.

If you don't have one and don't want to implement one - just pick the one we have in this repo and skip to [WebApp](#webapp) section.

### 1. Setup

1. Install dependencies

        cd wallet
        yarn

2. Configure environment

        cp .env.local.example .env.local
        editor-of-your-choice .env.local

3. Sign up Wallet Connect, create a new project and then set `REACT_APP_PROJECT_ID` in `.env.local` file

4. Start

        yarn start

### 2. Add Stacks support

As you can see, there's no Stacks wallet available on the list.

First of all we need to import Stacks libraries, second - configure a dummy account for us to use later.

> **REMEMBER** This is just a dummy wallet - don't use your "real" wallet credentials here!

1. Create Stacks library

    This file will do most of the work for us.

    It provides the wallet with methods used to send transactions, make contract calls and sign messages - this is the "core wallet" from Stacks' point of view. 

    The rest of code we'll add is just a boilerplate.

    Create `src/lib/StacksLib.ts` file:

        import { generateWallet, Wallet } from "@stacks/wallet-sdk";
        import { makeSTXTokenTransfer, makeContractCall, broadcastTransaction, AnchorMode } from '@stacks/transactions';
        import { StacksNetworkName } from '@stacks/network';
        import { BigNumber } from 'bignumber.js';

        import { toRealCV, toRealPostCondition } from "@web3devs/stacks-wallet-connect";

        /**
        * Types
        */
        interface IInitArguments {
            secretKey?: Uint8Array
        }

        /**
        * Library
        */
        export default class StacksLib {
            constructor() {}

            static init({ secretKey }: IInitArguments) {
                return new StacksLib()
            }

            public async getAddress() {
                return 'ST24YYAWQ4DK4RKCKK1RP4PX0X5SCSXTWQXFGVCVY'
            }

            public getSecretKey() {
                return 'oval mean brain hollow avoid battle year announce merge wing citizen north'
            }

            private async getWallet(): Promise<Wallet> {
                return await generateWallet({
                    secretKey: this.getSecretKey(),
                    password: '',
                });
            }

            public async signMessage(message: string) {
                return { signature: message + '+SIGNED' }
            }

            public async stxTransfer(params: any) {
                const wallet = await this.getWallet();

                const txOptions = {
                    recipient: params.recipient,
                    amount: new BigNumber(params.amount).times(1e6).toFixed(),
                    senderKey: wallet.accounts[0].stxPrivateKey,
                    network: 'testnet' as StacksNetworkName, // for mainnet, use 'mainnet'
                    anchorMode: AnchorMode.Any,
                };

                const transaction = await makeSTXTokenTransfer(txOptions);

                // to see the raw serialized tx
                const serializedTx = transaction.serialize().toString('hex');

                // broadcasting transaction to the specified network
                const broadcastResponse = await broadcastTransaction(transaction);
                const txId = broadcastResponse.txid;

                console.log('SIGN TXN: ', txId);
                return { txId }
            }

            public async contractCall(params: any) {
                const wallet = await generateWallet({
                    secretKey: this.getSecretKey(),
                    password: '',
                });

                const functionArgs = [];
                for (const arg of params.functionArgs) {
                    functionArgs.push(toRealCV(arg));
                }

                const postConditions = [];
                for (const pc of params.postConditions) {
                    postConditions.push(toRealPostCondition(pc));
                }

                const txOptions = {
                    senderKey: wallet.accounts[0].stxPrivateKey,
                    network: 'testnet' as StacksNetworkName, // for mainnet, use 'mainnet'
                    // validateWithAbi: true,
                    contractAddress: params.contractAddress,
                    contractName: params.contractName,
                    functionName: params.functionName,
                    functionArgs,
                    postConditions,
                    postConditionMode: params.postConditionMode,
                    anchorMode: AnchorMode.Any,
                };

                const transaction = await makeContractCall(txOptions);

                // to see the raw serialized tx
                const serializedTx = transaction.serialize().toString('hex');

                // broadcasting transaction to the specified network
                const broadcastResponse = await broadcastTransaction(transaction);
                const txId = broadcastResponse.txid;

                console.log('SIGN TXN: ', txId);
                return { txId }
            }
        }

2. Create data file to use with various UI elements `src/data/StacksData.ts`:

        import {
            STACKS_MAINNET_CHAIN_ID,
            STACKS_TESTNET_CHAIN_ID,
            STACKS_MAINNET_CHAIN_ID_PREFIXED,
            STACKS_TESTNET_CHAIN_ID_PREFIXED,
            StacksChainData,
        } from "@web3devs/stacks-wallet-connect";

        /**
        * Types
        */
        export type TStacksChain = keyof typeof STACKS_MAINNET_CHAINS

        /**
        * Chains
        */
        export const STACKS_MAINNET_CHAINS = {
            [STACKS_MAINNET_CHAIN_ID_PREFIXED]: {
                chainId: STACKS_MAINNET_CHAIN_ID,
                name: StacksChainData[STACKS_MAINNET_CHAIN_ID].name,
                logo: 'https://assets-global.website-files.com/618b0aafa4afde9048fe3926/6197e600ab7fc410890d1bc8_hiro.jpg',
                rgb: '33, 66, 99',
                rpc: ''
            }
        }

        export const STACKS_TEST_CHAINS = {
            [STACKS_TESTNET_CHAIN_ID_PREFIXED]: {
                chainId: STACKS_TESTNET_CHAIN_ID,
                name: StacksChainData[STACKS_TESTNET_CHAIN_ID].name,
                logo: 'https://assets-global.website-files.com/618b0aafa4afde9048fe3926/6197e600ab7fc410890d1bc8_hiro.jpg',
                rgb: '33, 66, 99',
                rpc: ''
            }
        }

        export const STACKS_CHAINS = { ...STACKS_MAINNET_CHAINS, ...STACKS_TEST_CHAINS }

3. Update helper file `src/utils/HelperUtil.ts`:

        ...

        import { STACKS_CHAINS, TStacksChain } from '@/data/StacksData'

        ...

        /**
         * Check if chain is part of STACKS standard
         */
        export function isStacksChain(chain: string) {
            return chain.includes('stacks')
        }

        ...

        export function formatChainName(chainId: string) {
            return (
                EIP155_CHAINS[chainId as TEIP155Chain]?.name ??
                COSMOS_MAINNET_CHAINS[chainId as TCosmosChain]?.name ??
                SOLANA_CHAINS[chainId as TSolanaChain]?.name ??
                STACKS_CHAINS[chainId as TStacksChain]?.name ?? // <== HERE
                chainId
            )
        }

4. Create utils lib `src/utils/StacksWalletUtil.ts`:

        import StacksLib from '@/lib/StacksLib'

        export let wallet1: StacksLib
        export let wallet2: StacksLib
        export let stacksWallets: Record<string, StacksLib>
        export let stacksAddresses: string[]

        let address1: string
        let address2: string

        /**
        * Utilities
        */
        export async function createOrRestoreStacksWallet() {
            const secretKey1 = localStorage.getItem('STACKS_SECRET_KEY_1')
            const secretKey2 = localStorage.getItem('STACKS_SECRET_KEY_2')

            if (secretKey1 && secretKey2) {
                const secretArray1: number[] = Object.values(JSON.parse(secretKey1))
                const secretArray2: number[] = Object.values(JSON.parse(secretKey2))
                wallet1 = StacksLib.init({ secretKey: Uint8Array.from(secretArray1) })
                wallet2 = StacksLib.init({ secretKey: Uint8Array.from(secretArray2) })
            } else {
                wallet1 = StacksLib.init({})
                wallet2 = StacksLib.init({})

                // // Don't store secretKey in local storage in a production project!
                // localStorage.setItem(
                //   'STACKS_SECRET_KEY_1',
                //   JSON.stringify(Array.from(wallet1.keypair.secretKey))
                // )
                // localStorage.setItem(
                //   'STACKS_SECRET_KEY_2',
                //   JSON.stringify(Array.from(wallet2.keypair.secretKey))
                // )
            }

            address1 = await wallet1.getAddress()
            address2 = await wallet2.getAddress()

            stacksWallets = {
                [address1]: wallet1,
                [address2]: wallet2
            }
            stacksAddresses = Object.keys(stacksWallets)

            return {
                stacksWallets,
                stacksAddresses
            }
        }

5. Update Settings Store `src/store/SettingsStore.ts`:

        ...
        interface State {
            ...
            stacksAddress: string
        }

        ...
        const state = proxy<State>({
            ...
            stacksAddress: ''
        })

        ...
        const SettingsStore = {
            ...
            setStacksAddress(stacksAddress: string) {
                state.stacksAddress = stacksAddress
            },
        ...


6. Update initialization `src/hooks/useInitialization.ts`:

        ...

        import { createOrRestoreStacksWallet } from '@/utils/StacksWalletUtil'

        ...

        const onInitialize = useCallback(async () => {
            try {
                const { eip155Addresses } = createOrRestoreEIP155Wallet()
                const { cosmosAddresses } = await createOrRestoreCosmosWallet()
                const { solanaAddresses } = await createOrRestoreSolanaWallet()
                const { stacksAddresses } = await createOrRestoreStacksWallet() // <=== HERE

                SettingsStore.setEIP155Address(eip155Addresses[0])
                SettingsStore.setCosmosAddress(cosmosAddresses[0])
                SettingsStore.setSolanaAddress(solanaAddresses[0])
                SettingsStore.setStacksAddress(stacksAddresses[0]) // <=== AND HERE

                await createSignClient()

                setInitialized(true)
            } catch (err: unknown) {
                alert(err)
            }
        }, [])


7. Create request handler util: `src/utils/StacksRequestHandlerUtil.ts`

    This file describes and integrates our Wallet API.

    > **NOTE** the import of `STACKS_DEFAULT_METHODS` - we'll use it later in the WebApp integration!

        import { getWalletAddressFromParams } from '@/utils/HelperUtil'
        import { stacksAddresses, stacksWallets } from '@/utils/StacksWalletUtil'
        import { formatJsonRpcError, formatJsonRpcResult } from '@json-rpc-tools/utils'
        import { SignClientTypes } from '@walletconnect/types'
        import { ERROR } from '@walletconnect/utils'

        import {
            STACKS_DEFAULT_METHODS
        } from "@web3devs/stacks-wallet-connect";

        export async function approveStacksRequest(
            requestEvent: SignClientTypes.EventArguments['session_request']
        ) {
            const { params, id } = requestEvent
            const { request } = params
            const wallet = stacksWallets[getWalletAddressFromParams(stacksAddresses, params)]

            switch (request.method) {
                case STACKS_DEFAULT_METHODS.SIGN_MESSAGE:
                const signedMessage = await wallet.signMessage(request.params.message)
                return formatJsonRpcResult(id, signedMessage)

                case STACKS_DEFAULT_METHODS.STX_TRANSFER:
                const stxTransferTxn = await wallet.stxTransfer(request.params)

                return formatJsonRpcResult(id, stxTransferTxn)

                case STACKS_DEFAULT_METHODS.CONTRACT_CALL:
                const contractCallTxn = await wallet.contractCall(request.params)

                return formatJsonRpcResult(id, contractCallTxn)

                default:
                throw new Error(ERROR.UNKNOWN_JSONRPC_METHOD.format().message)
            }
        }

        export function rejectStacksRequest(request: SignClientTypes.EventArguments['session_request']) {
            const { id } = request

            return formatJsonRpcError(id, ERROR.JSONRPC_REQUEST_METHOD_REJECTED.format().message)
        }

8. Add Stacks to the list of chains in `src/pages/index.tsx`:

        ...

        import { STACKS_MAINNET_CHAINS, STACKS_TEST_CHAINS } from '@/data/StacksData'

        ...

        export default function HomePage() {
            const {
                testNets,
                eip155Address,
                cosmosAddress,
                solanaAddress,
                stacksAddress // <== HERE
            } = useSnapshot(SettingsStore.state)
        ...

        <Text h4 css={{ marginBottom: '$5' }}>
            Mainnets
        </Text>

        ...

        {Object.values(STACKS_MAINNET_CHAINS).map(({ name, logo, rgb }) => (
            <AccountCard key={name} name={name} logo={logo} rgb={rgb} address={stacksAddress} />
        ))}

        ...

        <Text h4 css={{ marginBottom: '$5' }}>
            Testnets
        </Text>

        ...

        {Object.values(STACKS_TEST_CHAINS).map(({ name, logo, rgb }) => (
            <AccountCard key={name} name={name} logo={logo} rgb={rgb} address={stacksAddress} />
        ))}

        ...


    At this point you should see Stacks on the accounts list. If you have Stacks Wallet WebApp - you should be able to scan the QR code and see the connection confirmation window.

    The window doesn't work yet - we'll fix it now!

9. Add StacksData to `src/components/SessionProposalChainCard.tsx`:

        ...

        import { STACKS_MAINNET_CHAINS, STACKS_TEST_CHAINS } from '@/data/StacksData'

        ...

        const CHAIN_METADATA = {
        ...COSMOS_MAINNET_CHAINS,
        ...SOLANA_MAINNET_CHAINS,
        ...STACKS_MAINNET_CHAINS, // <== HERE
        ...EIP155_MAINNET_CHAINS,
        ...EIP155_TEST_CHAINS,
        ...SOLANA_TEST_CHAINS,
        ...STACKS_TEST_CHAINS // <== AND HERE
        }

        ...

10. Add StacksData to `src/components/SessionChainCard.tsx`:

    ...

    import { STACKS_MAINNET_CHAINS, STACKS_TEST_CHAINS } from '@/data/StacksData'

    ...

    const CHAIN_METADATA = {
    ...COSMOS_MAINNET_CHAINS,
    ...SOLANA_MAINNET_CHAINS,
    ...STACKS_MAINNET_CHAINS, // <=== HERE
    ...EIP155_MAINNET_CHAINS,
    ...EIP155_TEST_CHAINS,
    ...SOLANA_TEST_CHAINS,
    ...STACKS_TEST_CHAINS // <=== AND HERE
    }

    ...

11. Add Stacks to Session Proposal Modal `src/views/SessionProposalModal.tsx`:

        ...

        import {
            isCosmosChain,
            isEIP155Chain,
            isSolanaChain,
            isStacksChain // <=== HERE
        } from '@/utils/HelperUtil'

        ...

        import { stacksAddresses } from '@/utils/StacksWalletUtil'

        ...

        function renderAccountSelection(chain: string) {
        ...
            } else if (isStacksChain(chain)) {
                return (
                    <ProposalSelectSection
                        addresses={stacksAddresses}
                        selectedAddresses={selectedAccounts[chain]}
                        onSelect={onSelectAccount}
                        chain={chain}
                    />
                )
            }
        }

    At this point, you should be able to connect your web app to the wallet!

12. Exposing wallet's supported methods, modify `src/hooks/useWalletConnectEventsManager.ts`:

        ...

        import {
            STACKS_DEFAULT_METHODS
        } from "@web3devs/stacks-wallet-connect";

        ...

        const onSessionRequest = useCallback(
            async (requestEvent: SignClientTypes.EventArguments['session_request']) => {
                console.log('session_request', requestEvent)
                const { topic, params } = requestEvent
                const { request } = params
                const requestSession = signClient.session.get(topic)

                switch (request.method) {
                    ...

                    case STACKS_DEFAULT_METHODS.SIGN_MESSAGE:
                    case STACKS_DEFAULT_METHODS.STX_TRANSFER:
                    case STACKS_DEFAULT_METHODS.CONTRACT_CALL:
                        return ModalStore.open('SessionSignStacksModal', { requestEvent, requestSession })
        ...

13. Add `SessionSignStacksModal` to Modal Store `src/store/ModalStore.ts`:

        ...

        interface State {
            open: boolean
            view?:
                | 'SessionProposalModal'
                | 'SessionSignModal'
                | 'SessionSignTypedDataModal'
                | 'SessionSendTransactionModal'
                | 'SessionUnsuportedMethodModal'
                | 'SessionSignCosmosModal'
                | 'SessionSignSolanaModal'
                | 'SessionSignStacksModal' // <== HERE
            data?: ModalData
        }

14. Create the modal itself, add file `src/views/SessionSignStacksModal.tsx`:

        import ProjectInfoCard from '@/components/ProjectInfoCard'
        import RequestDataCard from '@/components/RequestDataCard'
        import RequesDetailsCard from '@/components/RequestDetalilsCard'
        import RequestMethodCard from '@/components/RequestMethodCard'
        import RequestModalContainer from '@/components/RequestModalContainer'
        import ModalStore from '@/store/ModalStore'
        import { approveStacksRequest, rejectStacksRequest } from '@/utils/StacksRequestHandlerUtil'
        import { signClient } from '@/utils/WalletConnectUtil'
        import { Button, Divider, Modal, Text } from '@nextui-org/react'
        import { Fragment } from 'react'

        export default function SessionSignStacksModal() {
            // Get request and wallet data from store
            const requestEvent = ModalStore.state.data?.requestEvent
            const requestSession = ModalStore.state.data?.requestSession

            // Ensure request and wallet are defined
            if (!requestEvent || !requestSession) {
                return <Text>Missing request data</Text>
            }

            // Get required request data
            const { topic, params } = requestEvent
            const { request, chainId } = params

            // Handle approve action (logic varies based on request method)
            async function onApprove() {
                if (requestEvent) {
                const response = await approveStacksRequest(requestEvent)
                await signClient.respond({
                    topic,
                    response
                })
                ModalStore.close()
                }
            }

            // Handle reject action
            async function onReject() {
                if (requestEvent) {
                const response = rejectStacksRequest(requestEvent)
                await signClient.respond({
                    topic,
                    response
                })
                ModalStore.close()
                }
            }

            return (
                <Fragment>
                <RequestModalContainer title="Sign Message">
                    <ProjectInfoCard metadata={requestSession.peer.metadata} />

                    <Divider y={2} />

                    <RequesDetailsCard chains={[chainId ?? '']} protocol={requestSession.relay.protocol} />

                    <Divider y={2} />

                    <RequestDataCard data={params} />

                    <Divider y={2} />

                    <RequestMethodCard methods={[request.method]} />
                </RequestModalContainer>

                <Modal.Footer>
                    <Button auto flat color="error" onClick={onReject}>
                    Reject
                    </Button>
                    <Button auto flat color="success" onClick={onApprove}>
                    Approve
                    </Button>
                </Modal.Footer>
                </Fragment>
            )
        }

15. Add the Stacks modal to Modal component `src/components/Modal.tsx`:

        ...

        import SessionSignStacksModal from '@/views/SessionSignStacksModal'

        ...

        return (
            <NextModal blur open={open} style={{ border: '1px solid rgba(139, 139, 139, 0.4)' }}>
                ...
                {view === 'SessionSignStacksModal' && <SessionSignStacksModal />}
            </NextModal>
        )

    That's it, your wallet should be operational now!

## WebApp

### 1. Setup

1. Install dependencies

        cd webapp
        yarn

2. Configure environment

        cp .env.local.example .env.local
        editor-of-your-choice .env.local

3. Sign up Wallet Connect, create a new project and then set `REACT_APP_PROJECT_ID` in `.env.local` file

4. Start

        yarn start

### 2. Add Stacks to the list of selectable chains

As you can see, there's no Stacks on the list of available chains. We need to add it there.

1. Import `@web3devs/stacks-wallet-connect`

        yarn add @web3devs/stacks-wallet-connect

2. Add `STACKS_MAINNET_CHAIN_ID_PREFIXED` and `STACKS_TESTNET_CHAIN_ID_PREFIXED` to `src/constants/default.ts`:

        import { STACKS_MAINNET_CHAIN_ID_PREFIXED, STACKS_TESTNET_CHAIN_ID_PREFIXED } from "@web3devs/stacks-wallet-connect"

        export const DEFAULT_MAIN_CHAINS = [
            // mainnets
            "eip155:1",
            "eip155:10",
            "eip155:100",
            "eip155:137",
            "eip155:42161",
            "eip155:42220",
            "cosmos:cosmoshub-4",
            "solana:4sGjMW1sUnHzSxGspuhpqLDx6wiyjNtZ",
            STACKS_MAINNET_CHAIN_ID_PREFIXED,
        ];

        export const DEFAULT_TEST_CHAINS = [
            // testnets
            "eip155:42",
            "eip155:69",
            "eip155:80001",
            "eip155:421611",
            "eip155:44787",
            "solana:8E9rvCKLFQia2Y35HXjjpWzj8weVo44K",
            STACKS_TESTNET_CHAIN_ID_PREFIXED
        ];

        ...

3. Add `StacksChainData` to `src/contexts/ChainDataContext.tsx`

        ...

        import { StacksChainData } from "@web3devs/stacks-wallet-connect"

        ...

        if (namespace === "solana") {
            chains = SolanaChainData;
        } else if (namespace === "stacks") { //ADD THIS
            chains = StacksChainData;
        } else {
            chains = await apiGetChainNamespace(namespace);
        }
        ...

4. Create `src/chains/stacks.ts`:

        import { NamespaceMetadata, ChainMetadata } from "../helpers";
        import { STACKS_MAINNET_CHAIN_ID, STACKS_TESTNET_CHAIN_ID } from "@web3devs/stacks-wallet-connect"

        export const StacksMetadata: NamespaceMetadata = {
            [STACKS_MAINNET_CHAIN_ID]: {
                logo: "https://assets-global.website-files.com/618b0aafa4afde65f2fe38fe/618b0aafa4afde2ae1fe3a1f_icon-isotipo.svg",
                rgb: "0, 0, 0",
            },
            [STACKS_TESTNET_CHAIN_ID]: {
                logo: "https://assets-global.website-files.com/618b0aafa4afde65f2fe38fe/618b0aafa4afde2ae1fe3a1f_icon-isotipo.svg",
                rgb: "0, 0, 0",
            },
        };

        export function getChainMetadata(chainId: string): ChainMetadata {
            const reference = chainId.split(":")[1];
            const metadata = StacksMetadata[reference];
            if (typeof metadata === "undefined") {
                throw new Error(`No chain metadata found for chainId: ${chainId}`);
            }
            return metadata;
        }

5. Import that file in `src/chains/index.tsdiff`

        ...

        import * as stacks from "./stacks";

        ...

        case "stacks":
            return stacks.getChainMetadata(chainId);

        ...

    At this point you should see Stacks logo on the list of available blockchains, but you still can't connect it.

6. Configuring wallet's API

    Each wallet in Wallet Connect provides interface (API) that can be consumed by clients.

    Everything so far was more or less - WebApp specific, but this is probably the most important bit of configuration.

    Let's assume the wallet we're connecting to specify 3 methods: 
    
    - `stacks_stxTransfer` for STX transfer
    - `stacks_contractCall` for contract interaction (ex. token transfer)
    - `stacks_signMessage` for signing messages

    Wallet Connect client will use these method when attempting to connect to a wallet:

        ...
        const requiredNamespaces = {
            "stacks": {
                "methods": [
                    "stacks_stxTransfer",
                    "stacks_contractCall",
                    "stacks_signMessage"
                ],
                "chains": [
                    "stacks:2147483648"
                ],
                "events": []
            }
        };

        const { uri, approval } = await client.connect({
          pairingTopic: pairing?.topic,
          requiredNamespaces,
        });

        if (uri) {
          QRCodeModal.open(uri, () => {
            console.log("EVENT", "QR Code Modal closed");
          });
        }

        const session = await approval();
        ...
    
    In our case, we need to add these to `src/helpers/namespaces.ts`:

        ...
        import { STACKS_DEFAULT_METHODS, STACKS_DEFAULT_EVENTS } from "@web3devs/stacks-wallet-connect"

        ...
        export const getSupportedMethodsByNamespace = (namespace: string) => {
            switch (namespace) {
                ...
                case "stacks":
                    return Object.values(STACKS_DEFAULT_METHODS);
                ...
            }
        };

        export const getSupportedEventsByNamespace = (namespace: string) => {
            switch (namespace) {
                ...
                    return Object.values(DEFAULT_SOLANA_EVENTS);
                case "stacks":
                    return Object.values(STACKS_DEFAULT_EVENTS);
                ...
            }
        };
    
    At this point you should be able to connect the app via Wallet Connect QR Code!

### 3. Add example calls to Wallet's API

So first, let's create Stacks RPC client:

1. Modify `src/contexts/JsonRpcContext.tsx` like this:

        ...
        // Import Stacks transactions stuff
        import {
            PostConditionMode,
            FungibleConditionCode,
        } from '@stacks/transactions'
        
        // Import stacks wallet connect stuff
        import {
            makeStandardFungiblePostCondition,
            createAssetInfo,
            uintCV,
            standardPrincipalCV,
            noneCV,
            STACKS_DEFAULT_METHODS
        } from "@web3devs/stacks-wallet-connect";

        ...

        //Modify IContext interface to include Stacks methods
        interface IContext {
            ping: () => Promise<void>;
            ....
            stacksRpc: {
                exampleSignMessage: TRpcRequestCallback;
                exampleStxTransfer: TRpcRequestCallback;
                exampleContractCall: TRpcRequestCallback;
            };
            ....
        }

        ...

        //Add the actual Stacks RPC client
        const stacksRpc = {
            //Contract call
            exampleContractCall: _createJsonRpcRequestHandler(
                async (chainId: string, address: string): Promise<IFormattedRpcResponse> => {
                    console.log("exampleContractCall", chainId, address, isTestnet);

                    const contract = 'ST24YYAWQ4DK4RKCKK1RP4PX0X5SCSXTWQXFGVCVY.fake-miamicoin-token-V2';
                    const contractPts = contract.split('.');
                    const contractAddress = contractPts[0];
                    const contractName = contractPts[1];
                    const tokenName = 'miamicoin'; //XXX: It's hidden in the contract's code but it's not hard to find.

                    const orderAmount = 13 * 10 ** 6;
                    const addressTo = 'ST3Q85SVTW7J3XQ38V7V88653YN90728NMM46J2ZE';

                    // Define post conditions
                    const postConditions = [];
                    postConditions.push(
                    makeStandardFungiblePostCondition(
                        address,
                        FungibleConditionCode.Equal,
                        orderAmount.toString(),
                        createAssetInfo(
                        contractAddress,
                        contractName,
                        tokenName
                        )
                    )
                    );

                    try {
                    const result = await client!.request<{ txId: string }>({
                        chainId,
                        topic: session!.topic,
                        request: {
                        method: STACKS_DEFAULT_METHODS.CONTRACT_CALL,
                        params: {
                            pubkey: address, //XXX: This one is required
                            postConditions,
                            contractAddress: contractAddress,
                            contractName: contractName,
                            functionName: 'transfer',
                            functionArgs: [
                                uintCV(orderAmount.toString()),
                                standardPrincipalCV(address),
                                standardPrincipalCV(addressTo),
                                noneCV(),
                            ],
                            // postConditionMode: PostConditionMode.Allow,
                            postConditionMode: PostConditionMode.Deny,
                        },
                        },
                    });

                    console.log('result', result);

                    return {
                        method: STACKS_DEFAULT_METHODS.CONTRACT_CALL,
                        address,
                        valid: true,
                        result: result.txId,
                    };
                    } catch (error: any) {
                    console.log('ERROR: ', error);
                    throw new Error(error);
                    }
                },
            ),

            //STX transfer
            exampleStxTransfer: _createJsonRpcRequestHandler(
                async (chainId: string, address: string): Promise<IFormattedRpcResponse> => {
                    console.log("exampleStxTransfer", chainId, address, isTestnet);

                    try {
                    const result = await client!.request<{ txId: string }>({
                        chainId,
                        topic: session!.topic,
                        request: {
                        method: STACKS_DEFAULT_METHODS.STX_TRANSFER,
                        params: {
                            pubkey: address, //XXX: This one is required
                            recipient: 'ST3Q85SVTW7J3XQ38V7V88653YN90728NMM46J2ZE',
                            amount: 12,
                        },
                        },
                    });

                    console.log('result', result);

                    return {
                        method: STACKS_DEFAULT_METHODS.STX_TRANSFER,
                        address,
                        valid: true,
                        result: result.txId,
                    };
                    } catch (error: any) {
                    console.log('ERROR: ', error);
                    throw new Error(error);
                    }
                },
            ),

            //Sign message
            exampleSignMessage: _createJsonRpcRequestHandler(
                async (chainId: string, address: string): Promise<IFormattedRpcResponse> => {
                    const message = 'loremipsum';

                    try {
                    const result = await client!.request<{ signature: string }>({
                        chainId,
                        topic: session!.topic,
                        request: {
                        method: STACKS_DEFAULT_METHODS.SIGN_MESSAGE,
                        params: {
                            pubkey: address, //XXX: This one is required
                            message,
                        },
                        },
                    });

                    //dummy check of signature
                    const valid = result.signature === message + '+SIGNED';

                    return {
                        method: STACKS_DEFAULT_METHODS.SIGN_MESSAGE,
                        address,
                        valid,
                        result: result.signature,
                    };
                    } catch (error: any) {
                    throw new Error(error);
                    }
                },
            ),
        };

        ...

        //Add it to context provider
        return (
            <JsonRpcContext.Provider
                value={{
                    ping,
                    ethereumRpc,
                    cosmosRpc,
                    solanaRpc,
                    stacksRpc, // <== HERE
                    rpcResult: result,
                    isRpcRequestPending: pending,
                    isTestnet,
                    setIsTestnet,
                }}
                >
                {children}
            </JsonRpcContext.Provider>
        );

    > **NOTICE** the use of `uintCV`, `standardPrincipalCV`, `standardPrincipalCV` etc. - these are **NOT** the functions you may be familiar with, but they share the same (or similar interface).

    > Wallet Connect serializes values through JSON.stringify and therefore Clarity Values can't be easily transfered over `client.request` interface.
    
    > We've provided the functions to ease development (the same or similar interface), but your wallet may use something else for that!

2. Let's use our newly created Stacks RPC `src/App.tsx` like this:

        ...

        import { STACKS_DEFAULT_METHODS } from "@web3devs/stacks-wallet-connect";

        ...

        //Import Stacks RPC
        const {
            ping,
            ethereumRpc,
            cosmosRpc,
            solanaRpc,
            stacksRpc, // <=== ADD THIS
            isRpcRequestPending,
            rpcResult,
            isTestnet,
            setIsTestnet,
        } = useJsonRpc();

        ...

        //Handle Stacks related actions
        const getStacksActions = (): AccountAction[] => {
            const onContractCall = async (chainId: string, address: string) => {
                openRequestModal();
                await stacksRpc.exampleContractCall(chainId, address);
            };

            const onStxTransfer = async (chainId: string, address: string) => {
                openRequestModal();
                await stacksRpc.exampleStxTransfer(chainId, address);
            };

            const onSignMessage = async (chainId: string, address: string) => {
                openRequestModal();
                await stacksRpc.exampleSignMessage(chainId, address);
            };

            return [
                { method: STACKS_DEFAULT_METHODS.CONTRACT_CALL, callback: onContractCall },
                { method: STACKS_DEFAULT_METHODS.STX_TRANSFER, callback: onStxTransfer },
                { method: STACKS_DEFAULT_METHODS.SIGN_MESSAGE, callback: onSignMessage },
            ];
        };

        ...

        //Add the actions
        const getBlockchainActions = (chainId: string) => {
            const [namespace] = chainId.split(":");
            switch (namespace) {
                ...
                case "stacks":
                    return getStacksActions();
                default:
                    break;
            }
        };

That's it