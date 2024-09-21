# collection_code

**Folder**
- Adapter
  Ex. ProductListAdapter
- Api
  - Api_Interface (interface)
  - ApiServiceCall
- Model
  Ex. ProductMasterListResponse
- Util

**Activity**
- ProductList

--------------------------------------------------------------------------------------------------------------

Api_Interface service = ApiServiceCall.getClient(this).create(Api_Interface.class);
Observable<ProductMasterListResponse> DataList = service.ProductMasterGetData();
DataList.subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Observer<ProductMasterListResponse>() {
                    @Override
                    public void onSubscribe(Disposable d) {
                        //When API Call Start
                    }

                    @Override
                    public void onNext(ProductMasterListResponse productMasterListResponse) {
                        //When API Response is Success
                        if (productMasterListResponse != null) {
                        }
                    }

                    @Override
                    public void onError(Throwable e) {
                        //When API Response is Error
                        if (e instanceof HttpException) {
                            HttpException error = (HttpException) e;
                            Response response = error.response();
                            try {
                                JSONObject jObjError = new JSONObject(response.errorBody().string());
                                String message = jObjError.getString("Message");
                            } catch (JSONException ex) {
                                ex.printStackTrace();
                            } catch (IOException ex) {
                                ex.printStackTrace();
                            }
                        }
                        if (WaitDialog.isShowing())
                            WaitDialog.dismiss();
                    }

                    @Override
                    public void onComplete() {
                        //When API Call End
                        WaitDialog.dismiss();
                    }
                });
