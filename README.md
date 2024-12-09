# pagination Firestore Compose
In This We are Going to Implement Pagination using Paging 3 Library,  and We are Gonna Paginate The List of Documents from Firebase Cloud Firestore.


first of all Add These Dependencies in Your ``` build.gradle.kts(Module :app) ``` File under Gradle Scripts.
```
// For Paging 3 Runtime
implementation("androidx.paging:paging-runtime-ktx:3.3.4")

// For Paging 3 Compose
implementation("androidx.paging:paging-compose:3.3.4")

```

### For Version Catelog, If You Want

Add These in Your ``` libs.versions.toml ``` File under Gradle Scripts.
```
[versions]
# Other Verions

# For Paging
paging3 = "3.3.4"


[libraries]
# Other Libraries

# For Paging
androidx-paging-runtime-ktx = { group = "androidx.paging", name = "paging-runtime-ktx", version.ref = "paging" }
androidx-paging-compose = { group = "androidx.paging", name = "paging-compose", version.ref = "paging" }

```

Then In You ``` build.gradle.kts(Module :app) ``` File.

```
// For Paging Runtime
implementation(libs.androidx.paging.runtime.ktx)

// For Paging Compose
implementation(libs.androidx.paging.compose)

```

## Steps

1. Creating Paging Source.
2. Repository Part.
3. ViewModel Part.
4. UI Part.

### 1. Paging Source
Paging Source is just a class.
```
class NewsListPagingSource(
    private val query: Query,
) : PagingSource<QuerySnapshot, NewsModel>() {

    override fun getRefreshKey(state: PagingState<QuerySnapshot, NewsModel>): QuerySnapshot? {
        return null
    }

    override suspend fun load(params: LoadParams<QuerySnapshot>): LoadResult<QuerySnapshot, NewsModel> {
        return try {

            delay(2000)

            val currentPage = params.key ?: query.get().await() // when the Last Item is null then we are getting the starting list

            val lastVisiblePage = currentPage.documents[currentPage.size() - 1] // Last Visible Document

            val nextPage = query.startAfter(lastVisiblePage).get().await() // Next Item after the Last Item


            return LoadResult.Page(
                data = currentPage.map {
                  it.toObject(NewsModel::class.java)
                },
                prevKey = null,
                nextKey = nextPage,
            )

        } catch (e: FirebaseFirestoreException) {
            LoadResult.Error(Throwable("Failed To load more news. May be cause of Internet!")) // Catching FirebaseFirestore Exception is Optional
        } catch (e: Exception) {
            return LoadResult.Error(e)
        }
    }

}

```
