# Collection Code

This repository contains the structure and codebase for an Android project that leverages Retrofit and RxJava for network operations, alongside a well-organized folder structure. Follow the steps below to set up and understand the project.

## Folder Structure

/collection_code ├── Adapter ├── Api ├── Model └── Util


- **Adapter**: Contains all adapters used in the app (e.g., RecyclerView adapters).
- **Api**: Handles network requests using Retrofit.
- **Model**: Contains the data model classes for handling responses.
- **Util**: Utility classes used across the project.

---

## Getting Started

### Prerequisites

Ensure that you have the following installed:

- Android Studio (latest stable version)
- Gradle (built-in with Android Studio)
- Java 8 or higher
- Internet connection (for downloading dependencies)

### Step 1: Add Dependencies

In your project’s `build.gradle` file, add the following dependencies for network operations using Retrofit and RxJava:

```groovy
dependencies {
    // RxJava for reactive programming
    implementation 'io.reactivex.rxjava2:rxandroid:2.1.1'
    implementation 'io.reactivex.rxjava2:rxjava:2.2.13'

    // Retrofit for networking
    implementation 'com.squareup.retrofit2:retrofit:2.8.1'
    implementation 'com.squareup.retrofit2:adapter-rxjava2:2.8.1'
    implementation 'com.squareup.retrofit2:converter-gson:2.8.1'

    // Logging Interceptor (optional, for debugging network calls)
    implementation 'com.squareup.okhttp3:logging-interceptor:4.4.1'
}

Sync your project with Gradle to download the required libraries.

Step 2: Create Folder Structure
Organize your project into the following folders:

Adapter: Place your RecyclerView or any other adapter classes here.
Api: This folder will contain all API-related code (e.g., interfaces and network service classes).
Model: Place your response model classes that map to JSON responses.
Util: Any utility classes can go here.
You can manually create this folder structure in Android Studio or through the file system.

Step 3: Add Files to Folders
Add the required files to each folder, as described below:

1. Model Folder
Define the data model for handling API responses. Here’s an example model class for a product list:
// Model: ProductMasterListResponse.java
public class ProductMasterListResponse {
    private List<Product> result;

    public List<Product> getResult() {
        return result;
    }

    public class Product {
        private String productName;
        private double defaultMRP;
        private double defaultSalePrice;
        private String imageUrl;
        private String stockStatus;

        // Getters and Setters...
    }
}

2. Api Folder
Create an interface for the API calls and a service class to handle network requests.
// Api_Interface.java
import io.reactivex.Observable;
import retrofit2.http.GET;

public interface Api_Interface {
    @GET("products")
    Observable<ProductMasterListResponse> getProductList();
}

// ApiServiceCall.java
import io.reactivex.android.schedulers.AndroidSchedulers;
import io.reactivex.schedulers.Schedulers;
import retrofit2.Retrofit;
import retrofit2.adapter.rxjava2.RxJava2CallAdapterFactory;
import retrofit2.converter.gson.GsonConverterFactory;

public class ApiServiceCall {
    private static final String BASE_URL = "https://your-api-url.com/";

    private static Retrofit retrofit = new Retrofit.Builder()
            .baseUrl(BASE_URL)
            .addConverterFactory(GsonConverterFactory.create())
            .addCallAdapterFactory(RxJava2CallAdapterFactory.create())
            .build();

    public static Api_Interface getApiInterface() {
        return retrofit.create(Api_Interface.class);
    }

    public static void fetchProductList(Callback callback) {
        getApiInterface().getProductList()
            .subscribeOn(Schedulers.io())
            .observeOn(AndroidSchedulers.mainThread())
            .subscribe(response -> callback.onSuccess(response),
                       throwable -> callback.onError(throwable));
    }

    public interface Callback {
        void onSuccess(ProductMasterListResponse response);
        void onError(Throwable error);
    }
}

3. Adapter Folder
Create the necessary adapters for your UI components. Below is an example RecyclerView adapter for displaying the product list.
// ProductMasterListAdapter.java
import android.content.Context;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.ImageView;
import android.widget.TextView;
import androidx.annotation.NonNull;
import androidx.recyclerview.widget.RecyclerView;
import com.bumptech.glide.Glide;
import java.util.List;

public class ProductMasterListAdapter extends RecyclerView.Adapter<ProductMasterListAdapter.ViewHolder> {
    private Context context;
    private List<ProductMasterListResponse.Product> productList;

    public ProductMasterListAdapter(Context context, List<ProductMasterListResponse.Product> productList) {
        this.context = context;
        this.productList = productList;
    }

    @NonNull
    @Override
    public ViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        View view = LayoutInflater.from(context).inflate(R.layout.item_product_list, parent, false);
        return new ViewHolder(view);
    }

    @Override
    public void onBindViewHolder(@NonNull ViewHolder holder, int position) {
        ProductMasterListResponse.Product product = productList.get(position);
        holder.productName.setText(product.getProductName());
        holder.salePrice.setText(String.format("$%.2f", product.getDefaultSalePrice()));

        Glide.with(context)
            .load(product.getImageUrl())
            .placeholder(R.drawable.ic_placeholder)
            .into(holder.productImage);
    }

    @Override
    public int getItemCount() {
        return productList.size();
    }

    public class ViewHolder extends RecyclerView.ViewHolder {
        TextView productName, salePrice;
        ImageView productImage;

        public ViewHolder(@NonNull View itemView) {
            super(itemView);
            productName = itemView.findViewById(R.id.txt_ProductName);
            salePrice = itemView.findViewById(R.id.txt_SalePrice);
            productImage = itemView.findViewById(R.id.img_ProductImage);
        }
    }
}

Step 4: Implement in Activity
In your activity, initialize the RecyclerView and Adapter, then fetch data from the API.
// ProductMaster.java
import android.os.Bundle;
import androidx.appcompat.app.AppCompatActivity;
import androidx.recyclerview.widget.LinearLayoutManager;
import androidx.recyclerview.widget.RecyclerView;
import java.util.List;

public class ProductMaster extends AppCompatActivity {
    private RecyclerView recyclerView;
    private ProductMasterListAdapter adapter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_product_master);

        recyclerView = findViewById(R.id.recycler_view);
        recyclerView.setLayoutManager(new LinearLayoutManager(this));

        fetchProductData();
    }

    private void fetchProductData() {
        ApiServiceCall.fetchProductList(new ApiServiceCall.Callback() {
            @Override
            public void onSuccess(ProductMasterListResponse response) {
                List<ProductMasterListResponse.Product> productList = response.getResult();
                adapter = new ProductMasterListAdapter(ProductMaster.this, productList);
                recyclerView.setAdapter(adapter);
            }

            @Override
            public void onError(Throwable error) {
                // Handle error
            }
        });
    }
}

Step 5: Run the App
Once everything is set up, you can run the app. The product list should be fetched from the API, displayed in a RecyclerView, and updated in real-time.

Additional Notes
Make sure to handle API keys and base URLs securely in a production environment.
You can modify the folder structure as per your project’s complexity.
For more detailed documentation on Retrofit and RxJava, please refer to their official sites.

Happy coding!

This comprehensive README file includes steps to set up the project, organize the folder structure, add dependencies, and create necessary files. It’s easy for other developers to follow and extend based on their project needs.
