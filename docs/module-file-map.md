# Module File Map

<!-- MODULE_FILE_MAP_START -->
app/cbl/COSGN00C.cbl = authentication
app/cpy/CSUSR01Y.cpy = authentication, user-management
app/cpy/COCOM01Y.cpy = authentication
app/bms/COSGN00.bms = authentication
app/cpy-bms/COSGN00.CPY = authentication

app/cbl/COACTVWC.cbl = accounts
app/cbl/COACTUPC.cbl = accounts
app/bms/COACTVW.bms = accounts
app/bms/COACTUP.bms = accounts
app/cpy-bms/COACTVW.CPY = accounts
app/cpy-bms/COACTUP.CPY = accounts
app/cpy/CVACT01Y.cpy = accounts, batch-processing
app/cpy/CVACT02Y.cpy = accounts, batch-processing
app/cpy/CVACT03Y.cpy = accounts, batch-processing
app/cpy/CVCUS01Y.cpy = accounts, batch-processing

app/cbl/COCRDLIC.cbl = credit-cards
app/cbl/COCRDSLC.cbl = credit-cards
app/cbl/COCRDUPC.cbl = credit-cards
app/bms/COCRDLI.bms = credit-cards
app/bms/COCRDSL.bms = credit-cards
app/bms/COCRDUP.bms = credit-cards
app/cpy-bms/COCRDLI.CPY = credit-cards
app/cpy-bms/COCRDSL.CPY = credit-cards
app/cpy-bms/COCRDUP.CPY = credit-cards
app/cpy/CVCRD01Y.cpy = credit-cards, authorization

app/cbl/COTRN00C.cbl = transactions
app/cbl/COTRN01C.cbl = transactions
app/cbl/COTRN02C.cbl = transactions
app/bms/COTRN00.bms = transactions
app/bms/COTRN01.bms = transactions
app/bms/COTRN02.bms = transactions
app/cpy-bms/COTRN00.CPY = transactions
app/cpy-bms/COTRN01.CPY = transactions
app/cpy-bms/COTRN02.CPY = transactions
app/cpy/CVTRA01Y.cpy = transactions, batch-processing
app/cpy/CVTRA02Y.cpy = transactions, batch-processing
app/cpy/CVTRA03Y.cpy = transactions, batch-processing
app/cpy/CVTRA04Y.cpy = transactions, batch-processing
app/cpy/CVTRA05Y.cpy = transactions, batch-processing
app/cpy/CVTRA06Y.cpy = transactions, batch-processing
app/cpy/CVTRA07Y.cpy = transactions, batch-processing

app/cbl/COBIL00C.cbl = bill-payment
app/bms/COBIL00.bms = bill-payment
app/cpy-bms/COBIL00.CPY = bill-payment

app/cbl/CORPT00C.cbl = reports
app/bms/CORPT00.bms = reports
app/cpy-bms/CORPT00.CPY = reports
app/cbl/CBSTM03A.CBL = reports, batch-processing
app/cbl/CBSTM03B.CBL = reports, batch-processing

app/cbl/COUSR00C.cbl = user-management
app/cbl/COUSR01C.cbl = user-management
app/cbl/COUSR02C.cbl = user-management
app/cbl/COUSR03C.cbl = user-management
app/bms/COUSR00.bms = user-management
app/bms/COUSR01.bms = user-management
app/bms/COUSR02.bms = user-management
app/bms/COUSR03.bms = user-management
app/cpy-bms/COUSR00.CPY = user-management
app/cpy-bms/COUSR01.CPY = user-management
app/cpy-bms/COUSR02.CPY = user-management
app/cpy-bms/COUSR03.CPY = user-management

app/cbl/COADM01C.cbl = user-management
app/bms/COADM01.bms = user-management
app/cpy-bms/COADM01.CPY = user-management
app/cpy/COADM02Y.cpy = user-management

app/cbl/CBACT01C.cbl = batch-processing
app/cbl/CBACT02C.cbl = batch-processing
app/cbl/CBACT03C.cbl = batch-processing
app/cbl/CBACT04C.cbl = batch-processing
app/cbl/CBTRN01C.cbl = batch-processing, transactions
app/cbl/CBTRN02C.cbl = batch-processing, transactions
app/cbl/CBTRN03C.cbl = batch-processing, transactions
app/cbl/CBCUS01C.cbl = batch-processing
app/cbl/CBEXPORT.cbl = batch-processing
app/cbl/CBIMPORT.cbl = batch-processing
app/cbl/CSUTLDTC.cbl = batch-processing
app/cbl/COBSWAIT.cbl = batch-processing
app/jcl/** = batch-processing
app/proc/** = batch-processing
app/scheduler/** = batch-processing
app/cpy/CSDAT01Y.cpy = batch-processing
app/cpy/CSUTLDPY.cpy = batch-processing
app/cpy/CSUTLDWY.cpy = batch-processing
app/cpy/CODATECN.cpy = batch-processing

app/cpy/COMEN02Y.cpy = authentication, accounts, credit-cards, transactions
app/cpy/COCOM01Y.cpy = authentication
app/cpy/COTTL01Y.cpy = authentication
app/cpy/CSMSG01Y.cpy = authentication
app/cpy/CSMSG02Y.cpy = authentication
app/cpy/CSLKPCDY.cpy = authentication
app/cpy/CSSETATY.cpy = authentication
app/cpy/CSSTRPFY.cpy = authentication
app/cpy/CVEXPORT.cpy = batch-processing

app/app-authorization-ims-db2-mq/cbl/COPAUA0C.cbl = authorization
app/app-authorization-ims-db2-mq/cbl/COPAUS0C.cbl = authorization
app/app-authorization-ims-db2-mq/cbl/COPAUS1C.cbl = authorization
app/app-authorization-ims-db2-mq/cbl/COPAUS2C.cbl = authorization
app/app-authorization-ims-db2-mq/cbl/CBPAUP0C.cbl = authorization, batch-processing
app/app-authorization-ims-db2-mq/cbl/DBUNLDGS.CBL = authorization, batch-processing
app/app-authorization-ims-db2-mq/cbl/PAUDBLOD.CBL = authorization, batch-processing
app/app-authorization-ims-db2-mq/cbl/PAUDBUNL.CBL = authorization, batch-processing
app/app-authorization-ims-db2-mq/bms/** = authorization
app/app-authorization-ims-db2-mq/cpy/** = authorization
app/app-authorization-ims-db2-mq/cpy-bms/** = authorization
app/app-authorization-ims-db2-mq/ims/** = authorization
app/app-authorization-ims-db2-mq/ddl/** = authorization
app/app-authorization-ims-db2-mq/dcl/** = authorization
app/app-authorization-ims-db2-mq/jcl/** = authorization, batch-processing

app/app-transaction-type-db2/cbl/COTRTLIC.cbl = transaction-type-db2
app/app-transaction-type-db2/cbl/COTRTUPC.cbl = transaction-type-db2
app/app-transaction-type-db2/cbl/COBTUPDT.cbl = transaction-type-db2, batch-processing
app/app-transaction-type-db2/bms/** = transaction-type-db2
app/app-transaction-type-db2/cpy/** = transaction-type-db2
app/app-transaction-type-db2/cpy-bms/** = transaction-type-db2
app/app-transaction-type-db2/ddl/** = transaction-type-db2
app/app-transaction-type-db2/dcl/** = transaction-type-db2
app/app-transaction-type-db2/jcl/** = transaction-type-db2, batch-processing

app/app-vsam-mq/cbl/COACCT01.cbl = mq-integration, accounts
app/app-vsam-mq/cbl/CODATE01.cbl = mq-integration
app/app-vsam-mq/csd/** = mq-integration

app/asm/** = batch-processing
app/maclib/** = batch-processing
app/catlg/** = batch-processing
app/data/** = batch-processing
app/csd/CARDDEMO.CSD = authentication
<!-- MODULE_FILE_MAP_END -->
