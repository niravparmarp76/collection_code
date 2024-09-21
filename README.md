# collection_code

**Folder**
- Adapter
- Api
  - Api_Interface
  - ApiServiceCall
- Model
- Util

**Step : 1** Add dependencies

// retrofit
implementation 'io.reactivex.rxjava2:rxandroid:2.1.1'
implementation 'io.reactivex.rxjava2:rxjava:2.2.13'
implementation 'com.squareup.retrofit2:retrofit:2.8.1'
implementation 'com.squareup.retrofit2:adapter-rxjava2:2.8.1'
implementation 'com.squareup.retrofit2:converter-gson:2.8.1'
implementation 'com.squareup.okhttp3:logging-interceptor:4.4.1'

**Step : 2** Create above folder structure

**Step : 3** Created folder structure in add below file 

Model -> Ex. ProductMasterListResponse
Api -> Api_Interface.java and ApiServiceCall.java
Adapter -> Ex. ProductMasterListAdapter
ProductMaster.java
- activity_product_master.xml
