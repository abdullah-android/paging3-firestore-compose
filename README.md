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
