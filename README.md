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

--------------------------------------------------------------------------------------------------------------
ProductList.java
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


-------------------------------------------------------------------------------------------------------------------------------
ApiServiceCall.java
-------------------------------------------------------------------------------------------------------------------------------

package application.perk.merchant.Api;

import android.content.Context;

import com.google.gson.Gson;
import com.google.gson.GsonBuilder;

import java.util.concurrent.TimeUnit;

import application.perk.merchant.BuildConfig;
import okhttp3.OkHttpClient;
import okhttp3.logging.HttpLoggingInterceptor;
import retrofit2.Retrofit;
import retrofit2.adapter.rxjava2.RxJava2CallAdapterFactory;
import retrofit2.converter.gson.GsonConverterFactory;

public class ApiServiceCall {
    public static String BASE_URL = Resources.getSystem().getString(R.string.api_url);

    private static Retrofit retrofit = null;

    public static Retrofit getClient(Context context) {
//        OkHttpClient okHttpClient = HttpClient.getOkHttpClient();
        HttpLoggingInterceptor logging = new HttpLoggingInterceptor();

        if (BuildConfig.DEBUG) {
            logging.setLevel(HttpLoggingInterceptor.Level.BODY);
        } else {
            logging.setLevel(HttpLoggingInterceptor.Level.NONE);
        }

        OkHttpClient okHttpClient = new OkHttpClient.Builder()
                .connectTimeout(120, TimeUnit.SECONDS)
                .readTimeout(30, TimeUnit.SECONDS)
                .addInterceptor(logging).build();

        Gson gson = new GsonBuilder()
                .setDateFormat("yyyy-MM-dd'T'HH:mm:ssZ")
                .create();
        if (retrofit == null) {
            retrofit = new Retrofit.Builder()
                    .baseUrl(BASE_URL)
                    .addConverterFactory(GsonConverterFactory.create(gson))
                    .addCallAdapterFactory(RxJava2CallAdapterFactory.create())
                    .client(okHttpClient)
                    .build();
        }
        return retrofit;
    }
}

------------------------------------------------------------------------------------------------------------------------
Api_Interface.java
-------------------------------------------------------------------------------------------------------------------------------

package application.perk.merchant.Api;

import application.perk.merchant.Model.ProductMasterListResponse;
import io.reactivex.Observable;
import retrofit2.http.Body;
import retrofit2.http.Field;
import retrofit2.http.FormUrlEncoded;
import retrofit2.http.POST;

public interface Api_Interface {
    @FormUrlEncoded
    @POST("ProductMaster/GetData")
    Observable<ProductMasterListResponse> ProductMasterGetData(@Field("StoreCode") String StoreCode
            , @Field("ProductID") int ProductID
            , @Field("isActive") int isActive
            , @Field("PageNo") int PageNo
            , @Field("RecordCount") int RecordCount
    );
}

-------------------------------------------------------------------------------------------------------------------
ProductMasterListResponse.java
-------------------------------------------------------------------------------------------------------------------------------

package application.perk.merchant.Model;

import java.util.List;

public class ProductMasterListResponse {
    int Status;
    String Message;
    int TotalPageCount;
    int TotalRecordCount;
    result Result;

    public int getStatus() {
        return Status;
    }

    public String getMessage() {
        return Message;
    }

    public int getTotalPageCount() {
        return TotalPageCount;
    }

    public int getTotalRecordCount() {
        return TotalRecordCount;
    }

    public result getResult() {
        return Result;
    }

    public class result {
        List<table> Table;

        public List<table> getTable() {
            return Table;
        }

        public class table {
            int ProductID;
            String ProductSKU;
            String ProductName;
            float DefaultSalePrice;
            float DefaultMRP;
            String Description;
            String CountryOfOrigin;
            String ProductLinks;
            String StockStatus;
            float Ratings;
            int isActive;
            String ImageURL;
            int ProductGroupID;

            public int getProductID() {
                return ProductID;
            }

            public String getProductSKU() {
                return ProductSKU;
            }

            public String getProductName() {
                return ProductName;
            }

            public float getDefaultSalePrice() {
                return DefaultSalePrice;
            }

            public float getDefaultMRP() {
                return DefaultMRP;
            }

            public String getDescription() {
                return Description;
            }

            public String getCountryOfOrigin() {
                return CountryOfOrigin;
            }

            public String getProductLinks() {
                return ProductLinks;
            }

            public String getStockStatus() {
                return StockStatus;
            }

            public float getRatings() {
                return Ratings;
            }

            public int getisActive() {
                return isActive;
            }

            public String getImageURL() {
                return ImageURL;
            }

            public int getProductGroupID() {
                return ProductGroupID;
            }
        }
    }
}
