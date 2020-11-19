##############################################################
WeBankBlockchain-Gov 技术文档
##############################################################

.. admonition:: 什么是 WeBankBlockchain-Gov 

    WeBankBlockchain-Gov 是微众银行立足于分布式协作的联盟链场景，提炼和解决了常见的痛点和问题，所打造的一系列简单好用的组件，助力区块链社区、生态和产品协调发展。

    WeBankBlockchain-Gov 是一套稳定、高效、安全的区块链数据治理组件解决方案，可无缝适配FISCO BCOS区块链底层平台。
    首批开源的有账户治理组件(WeBankBlockchain-Gov-Acct)、合约权限服务组件(Data-Stash)、 私钥管理组件(Data-Reconcile)和证书管理组件（WeBankBlockchain-Gov-Cert）。

    这四个组件分别从私钥丢失重置、合约权限细粒度管控、私钥和证书的全生命周期管控等方面着手，提供了可部署的智能合约代码、易于使用的SDK和可参考的落地实践Demo等交付物。
    我们将竭尽所能在实践中持续探索和开发新的治理组件，也希望社区的朋友们一起加入进来，不断为WeBankBlockchain-Gov补充新鲜血液。

.. admonition:: 设计目标

    随着区块链技术的不断发展和应用的加速落地，区块链参与者不再仅仅只关注于共识算法选择、性能扩展等技术问题，也开始关注对等协作的参与者如何解决分歧、达成共识、多方共治。

    WeBankBlockchain-Gov 提出了轻量解耦、通用场景、一站式、简洁易用的四大目标。

        - **轻量解耦**。所有的治理组件与具体的业务解耦。可轻量化集成，在不侵入底层的前提下可插拔。通过类库、智能合约、SDK等多种方式提供。使用者甚至只需要使用链控制台，就可以部署和管控治理过程。
    
        - **通用场景**。所有治理组件所瞄准的都是所有联盟链治理中的“刚需”场景，例如首批开源的账户重置、合约权限、私钥和证书的生命周期管理，账户、合约、私钥和证书堪称联盟链技术及上层治理的基石。
    
        - **一站式**。正所谓“一件在手，天下我有”。对于每种组件，致力于覆盖该领域几乎所有主流场景。以私钥管理组件为例，支持多种私钥生成方式和格式、覆盖几乎所有主流场景，提供基于文件、多数据库等托管方式，并支持私钥派生、分片等加密方式。
        
        - **简洁易用**。致力于提供傻瓜式的使用体验。

    WeBankBlockchain-Data 定位为区块链链治理组件，不仅希望在开发层面提供趁手的工具，更希望在实践层面为区块链参与者提供可参考落地的实践案例，从整体上助力区块链行业治理水平的提升。



.. admonition:: 组件简介

    - **WeBankBlockchain-Gov-Acct  账户治理组件** 
    基于智能合约开发，提供区块链用户账户注册、私钥重置、冻结、解冻等账户全生命周期管理，支持管理员、阈值投票、多签制等多种治理策略。
    请参考 `文档 <./docs/WeBankBlockchain-Gov-Acct/index.html>`_
    
    - **WeBankBlockchain-Gov-Auth  合约权限服务** 
    基于智能合约，提供区块链账户、合约、函数等粒度的权限控制的功能的通用组件。
    请参考 `文档 <./docs/WeBankBlockchain-Gov-Auth/index.html>`_ 
    
    - **WeBankBlockchain-Gov-Key  私钥管理组件**
    提供私钥生成、存储、加解密、加签、验签等私钥全生命周期管理的通用解决方案。
    请参考 `文档 <./docs/WeBankBlockchain-Gov-Key/index.html>`_

    - **WeBankBlockchain-Gov-Cert  证书管理组件**
    提供证书生成、验证、子证书请求等证书全生命周期管理的通用解决方案。
    灵活配置、无需开发，支持自定义对账数据和对账格式，支持定时对账和触发对账，对账处理模块可插拔可扩展。
    请参考 `文档 <./docs/WeBankBlockchain-Gov-Cert/index.html>`_



.. toctree::
   :maxdepth: 3

   ./docs/WeBankBlockchain-Gov-Acct/index.md
   ./docs/WeBankBlockchain-Gov-Auth/index.md
   ./docs/WeBankBlockchain-Gov-Key/index.md
   ./docs/WeBankBlockchain-Gov-Cert/index.md
.. 


 
 