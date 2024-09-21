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

    private void FillDataList() {
        Api_Interface service = ApiServiceCall.getClient(this).create(Api_Interface.class);
        Observable<ProductMasterListResponse> DataList = service.ProductMasterGetData(
                getResources().getString(R.string.StoreCode)
                , 0
                , DisplayType
                , CurrentPage
                , RecordCount);
        DataList.subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Observer<ProductMasterListResponse>() {
                    @Override
                    public void onSubscribe(Disposable d) {
                        //When API Call Start
                        WaitDialog.show();
                    }

                    @Override
                    public void onNext(ProductMasterListResponse productMasterListResponse) {
                        //When API Response is Success
                        Log.d(TAG, "Fill_ProductList onNext: ");
                        if (productMasterListResponse != null) {
                            if (productMasterListResponse.getStatus() == 200) {
                                if (productMasterListResponse.getTotalRecordCount() > 0) {
                                    lnr_NoDataFound.setVisibility(View.GONE);
                                    rv_DataList.setVisibility(View.VISIBLE);

                                    TotalPageCount = productMasterListResponse.getTotalPageCount();

                                    if (CurrentPage == 1) {
                                        //Recycler view Bind
                                        rv_DataList.setLayoutManager(new LinearLayoutManager(ProductList.this));
                                        ObjProductMasterListResponse.clear();
                                        ObjProductMasterListResponse.addAll(productMasterListResponse.getResult().getTable());
                                        ObjProductListAdapter = new ProductListAdapter(ProductList.this, ObjProductMasterListResponse, ProductList.this);
                                        rv_DataList.setAdapter(ObjProductListAdapter);
                                    } else {
                                        // Add new items to the list
                                        int startPosition = ObjProductMasterListResponse.size();
                                        ObjProductMasterListResponse.addAll(productMasterListResponse.getResult().getTable());
                                        ObjProductListAdapter.notifyItemRangeInserted(startPosition, productMasterListResponse.getResult().getTable().size());
                                        isLoading = false;
                                    }
                                }
                            }
                        }
                    }

                    @Override
                    public void onError(Throwable e) {
                        //When API Response is Error
                        Log.d(TAG, "FillDataList onError: " + e.toString());
                        if (e instanceof HttpException) {
                            HttpException error = (HttpException) e;
                            Response response = error.response();
                            try {
                                JSONObject jObjError = new JSONObject(response.errorBody().string());
                                String message = jObjError.getString("Message");
                                Log.d("LoadDefaultData", "onError: " + message);
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
    }
