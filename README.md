[![Documentation](https://docs.rs/reqwest_cookie_store/badge.svg)](https://docs.rs/reqwest_cookie_store)

`reqwest_cookie_store` provides implementations of `reqwest::cookie::CookieStore` for [cookie_store](https://crates.io/crates/cookie_store).

This crate is not directly associated with the `reqwest` project, and only provides a convenience implementation for utilizing
[cookie_store](https://crates.io/crates/cookie_store) with `reqwest`'s traits.

# Example
The following example demonstrates loading a `cookie_store::CookieStore` (re-exported in this crate) from disk, and using it within a
`CookieStoreMutex`. It then makes a series of requests, examining and modifying the contents
of the underlying `cookie_store::CookieStore` in between.

```rust
// Load an existing set of cookies, serialized as json, if it is available
let cookie_store = {
  if let Ok(file) = std::fs::File::open("cookies.json")
    .map(std::io::BufReader::new) {
      cookie_store::serde::json::load(file).unwrap()
    }
    else {
      reqwest_cookie_store::CookieStore::new(None)
    }
};
let cookie_store = reqwest_cookie_store::CookieStoreMutex::new(cookie_store);
let cookie_store = std::sync::Arc::new(cookie_store);
{
  // Examine initial contents
  println!("initial load");
  let store = cookie_store.lock().unwrap();
  for c in store.iter_any() {
    println!("{:?}", c);
  }
}
//!
// Build a `reqwest` Client, providing the deserialized store
let client = reqwest::Client::builder()
    .cookie_provider(std::sync::Arc::clone(&cookie_store))
    .build()
    .unwrap();
//!
// Make a sample request
client.get("https://google.com").send().await.unwrap();
{
  // Examine the contents of the store.
  println!("after google.com GET");
  let store = cookie_store.lock().unwrap();
  for c in store.iter_any() {
    println!("{:?}", c);
  }
}
//!
// Make another request from another domain
println!("GET from msn");
client.get("https://msn.com").send().await.unwrap();
{
  // Examine the contents of the store.
  println!("after msn.com GET");
  let mut store = cookie_store.lock().unwrap();
  for c in store.iter_any() {
    println!("{:?}", c);
  }
  // Clear the store, and examine again
  store.clear();
  println!("after clear");
  for c in store.iter_any() {
    println!("{:?}", c);
  }
}
//!
// Get some new cookies
client.get("https://google.com").send().await.unwrap();
{
  // Show serialized contents of the store. Alternatively, using
  // a `std::fs::File` for `writer` could be used to serialize
  // the store to disk:
  // let mut writer = std::fs::File::create("output_cookies.json")
  //     .map(std::io::BufWriter::new)
  //     .unwrap();
  let mut writer = std::io::BufWriter::new(Vec::new());
  let store = cookie_store.lock().unwrap();
  cookie_store::serde::json::save(&store, &mut writer).unwrap();
  let cookies_json = String::from_utf8(writer.into_inner().unwrap()).unwrap();
  println!("JSON serialization of cookies contents: {cookies_json}");
  assert!(cookies_json.contains("Domain=google.com"));
}
```
