# Module File Map

<!-- MODULE_FILE_MAP_START -->
app/cbl/COSGN00C.cbl = signon
app/bms/COSGN00.bms = signon
app/cpy-bms/COSGN00.CPY = signon
app/cbl/COMEN01C.cbl = signon
app/bms/COMEN01.bms = signon
app/cpy-bms/COMEN01.CPY = signon
app/cpy/CSUSR01Y.cpy = signon, user-management

app/cbl/COACTVWC.cbl = account-management
app/cbl/COACTUPC.cbl = account-management
app/bms/COACTVW.bms = account-management
app/bms/COACTUP.bms = account-management
app/cpy-bms/COACTVW.CPY = account-management
app/cpy-bms/COACTUP.CPY = account-management
app/cpy/CVACT01Y.cpy = account-management, batch-processing
app/cpy/CVCUS01Y.cpy = account-management, batch-processing

app/cbl/COCRDLIC.cbl = card-management
app/cbl/COCRDSLC.cbl = card-management
app/cbl/COCRDUPC.cbl = card-management
app/bms/COCRDLI.bms = card-management
app/bms/COCRDSL.bms = card-management
app/bms/COCRDUP.bms = card-management
app/cpy-bms/COCRDLI.CPY = card-management
app/cpy-bms/COCRDSL.CPY = card-management
app/cpy-bms/COCRDUP.CPY = card-management
app/cpy/CVACT02Y.cpy = card-management, batch-processing
app/cpy/CVACT03Y.cpy = card-management, batch-processing, transaction-processing

app/cbl/COTRN00C.cbl = transaction-processing
app/cbl/COTRN01C.cbl = transaction-processing
app/cbl/COTRN02C.cbl = transaction-processing
app/cbl/CORPT00C.cbl = transaction-processing
app/bms/COTRN00.bms = transaction-processing
app/bms/COTRN01.bms = transaction-processing
app/bms/COTRN02.bms = transaction-processing
app/bms/CORPT00.bms = transaction-processing
app/cpy-bms/COTRN00.CPY = transaction-processing
app/cpy-bms/COTRN01.CPY = transaction-processing
app/cpy-bms/COTRN02.CPY = transaction-processing
app/cpy-bms/CORPT00.CPY = transaction-processing
app/cpy/CVTRA05Y.cpy = transaction-processing, batch-processing
app/cpy/CVTRA06Y.cpy = transaction-processing, batch-processing
app/cpy/CVTRA03Y.cpy = transaction-processing, batch-processing, transaction-type-management
app/cpy/CVTRA04Y.cpy = transaction-processing, batch-processing, transaction-type-management

app/cbl/COBIL00C.cbl = bill-payment
app/bms/COBIL00.bms = bill-payment
app/cpy-bms/COBIL00.CPY = bill-payment

app/cbl/COUSR00C.cbl = user-management
app/cbl/COUSR01C.cbl = user-management
app/cbl/COUSR02C.cbl = user-management
app/cbl/COUSR03C.cbl = user-management
app/cbl/COADM01C.cbl = user-management
app/bms/COUSR00.bms = user-management
app/bms/COUSR01.bms = user-management
app/bms/COUSR02.bms = user-management
app/bms/COUSR03.bms = user-management
app/bms/COADM01.bms = user-management
app/cpy-bms/COUSR00.CPY = user-management
app/cpy-bms/COUSR01.CPY = user-management
app/cpy-bms/COUSR02.CPY = user-management
app/cpy-bms/COUSR03.CPY = user-management
app/cpy-bms/COADM01.CPY = user-management

app/cbl/CBTRN01C.cbl = batch-processing
app/cbl/CBTRN02C.cbl = batch-processing
app/cbl/CBTRN03C.cbl = batch-processing, transaction-processing
app/cbl/CBACT01C.cbl = batch-processing, account-management
app/cbl/CBACT02C.cbl = batch-processing, account-management
app/cbl/CBACT03C.cbl = batch-processing, account-management
app/cbl/CBACT04C.cbl = batch-processing
app/cbl/CBSTM03A.CBL = batch-processing
app/cbl/CBSTM03B.CBL = batch-processing
app/cbl/CBEXPORT.cbl = batch-processing
app/cbl/CBIMPORT.cbl = batch-processing
app/cbl/CBCUS01C.cbl = batch-processing, account-management
app/cbl/CSUTLDTC.cbl = batch-processing
app/cbl/COBSWAIT.cbl = batch-processing
app/cpy/CVTRA01Y.cpy = batch-processing
app/cpy/CVTRA02Y.cpy = batch-processing
app/cpy/COADM02Y.cpy = batch-processing, user-management
app/cpy/COCOM01Y.cpy = batch-processing
app/cpy/CODATECN.cpy = batch-processing
app/cpy/COMEN02Y.cpy = batch-processing
app/cpy/COSTM01.CPY = batch-processing
app/cpy/COTTL01Y.cpy = batch-processing
app/cpy/CSDAT01Y.cpy = batch-processing
app/cpy/CSLKPCDY.cpy = batch-processing
app/cpy/CSMSG01Y.cpy = batch-processing
app/cpy/CSMSG02Y.cpy = batch-processing
app/cpy/CSSETATY.cpy = batch-processing
app/cpy/CSSTRPFY.cpy = batch-processing
app/cpy/CSUTLDPY.cpy = batch-processing
app/cpy/CSUTLDWY.cpy = batch-processing
app/cpy/CUSTREC.cpy = batch-processing, account-management
app/cpy/CVEXPORT.cpy = batch-processing
app/jcl/** = batch-processing
app/proc/** = batch-processing
app/asm/** = batch-processing
app/maclib/** = batch-processing
app/scheduler/** = batch-processing

app/app-authorization-ims-db2-mq/cbl/** = authorization
app/app-authorization-ims-db2-mq/bms/** = authorization
app/app-authorization-ims-db2-mq/cpy/** = authorization
app/app-authorization-ims-db2-mq/cpy-bms/** = authorization
app/app-authorization-ims-db2-mq/ims/** = authorization
app/app-authorization-ims-db2-mq/ddl/** = authorization
app/app-authorization-ims-db2-mq/dcl/** = authorization
app/app-authorization-ims-db2-mq/jcl/** = authorization
app/app-authorization-ims-db2-mq/data/** = authorization

app/app-transaction-type-db2/cbl/** = transaction-type-management
app/app-transaction-type-db2/bms/** = transaction-type-management
app/app-transaction-type-db2/cpy/** = transaction-type-management
app/app-transaction-type-db2/cpy-bms/** = transaction-type-management
app/app-transaction-type-db2/ddl/** = transaction-type-management
app/app-transaction-type-db2/dcl/** = transaction-type-management
app/app-transaction-type-db2/jcl/** = transaction-type-management
app/app-transaction-type-db2/ctl/** = transaction-type-management

app/app-vsam-mq/cbl/** = mq-integration
app/app-vsam-mq/csd/** = mq-integration
<!-- MODULE_FILE_MAP_END -->
