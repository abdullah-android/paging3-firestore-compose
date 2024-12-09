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

            // when the Last Item is null then we are getting the starting list
            val currentPage = params.key ?: query.get().await()

            // Last Visible Document
            val lastVisiblePage = currentPage.documents[currentPage.size() - 1]

             // Next Item after the Last Item
            val nextPage = query.startAfter(lastVisiblePage).get().await()


            return LoadResult.Page(
                data = currentPage.map {
                  it.toObject(NewsModel::class.java)
                },
                prevKey = null,
                nextKey = nextPage,
            )

        } catch (e: FirebaseFirestoreException) {
            // Catching FirebaseFirestore Exception is Optional
            LoadResult.Error(Throwable("Failed To load more news. May be cause of Internet!")) 
        } catch (e: Exception) {
            return LoadResult.Error(e)
        }
    }

}

```



### 2. Repository Part

In the Repository Interface

```
interface NewsRepository {

    // News Collection Refference
    val newsColRef: CollectionReference? 

    // This Returns the Flow of Paging Data of NewsModel, NewsModel is a data class
    fun getNewsList(): Flow<PagingData<NewsModel>> 

}

```

In Repository Implementation

```
class NewsRepositoryImpl(
    private val firestore: FirebaseFirestore,
) : NewsRepository {

    // References
    override val newsColRef: CollectionReference?
        get() = firestore.collection(FirestoreNodes.NEWS_COL)


    // News
    override fun getNewsList(category: String?): Flow<PagingData<NewsModel>> {

        val query = newsColRef
            ?.orderBy("timestamp", Query.Direction.DESCENDING)
            ?.limit(10)

        return Pager(
            config = PagingConfig(
                pageSize = 10,
            ),
            pagingSourceFactory = {
                NewsListPagingSource(
                    query!! // !! is for Non Null Query
                )
            }
        ).flow // We Get the flow that we want

    }

}

```




### 3. ViewModel Part

```
class NewsViewModel(
    private val newsRepository: NewsRepository,
) : ViewModel() {

    private val _newsList = MutableStateFlow<PagingData<NewsModel>>(PagingData.empty())
    val newsList: StateFlow<PagingData<NewsModel>> = _newsList

    init {
        getNewsList()
    }

    fun getNewsList() {
        viewModelScope.launch {
            newsRepository
                .getNewsList()
                .cachedIn(this)
                .collectLatest { pagingData ->
                    _newsList.value = pagingData
                }
        }
    }

}

```



### 4. UI Part

```

@Composable
fun NewsListScreen(
    newsViewModel: NewsViewModel = koinViewModel()  // Best Practice is using Dependency Injection, like koin or dagger hilt
) {

    val newsList = newsViewModel.newsList.collectAsLazyPagingItems()

    Scafold(
        modifier =  Modifier
            .fillMaxSize()
    ) { innerPadding ->
        LazyColumn(
            modifier = Modifier
                .fillMaxSize()
                .padding(paaddingValues = innerPadding)
        ) {
            items(
                count = newsList.itemCount,
                key = newsList.itemKey {
                    it.newsId ?: "
                }
            ) { index ->
                val item = newsList[index]

                NewsItem(item = item)
            }
        }
    }

}

```
