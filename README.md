# Substrate进阶课第4课作业

1. 完成在Offchain Worker中，使用Offchain Indexing特性实现从链上向Offchain Storage中写入数据

```rust
const ONCHAIN_TX_KEY: &[u8] = b"my_pallet::indexing1";
#[derive(Debug, Encode, Decode, Default)]
struct IndexingData(Vec<u8>, Vec<u8>);
#[pallet::weight(0)]
pub fn submit_data(origin: OriginFor<T>, payload: Vec<u8>) -> DispatchResultWithPostInfo {
	let who = ensure_signed(origin)?;

	log::info!("who:{:?} sign in submit_data call: {:?}", who, payload);

	Self::write_to_offchain_storage(payload);

	Ok(().into())
}

fn write_to_offchain_storage(payload: Vec<u8>) {
	let key = Self::derived_key(frame_system::Pallet::<T>::block_number());
	let data = IndexingData(b"submit_number_unsigned".to_vec(), payload);
	offchain_index::set(&key, &data.encode());
	log::info!("write to local storage by key: 0x{}, value: 0x{}", HexDisplay::from(&key), data);
}

fn derived_key(block_number: T::BlockNumber) -> Vec<u8> {
	block_number.using_encoded(|encoded_bn| {
		ONCHAIN_TX_KEY
			.clone()
			.into_iter()
			.chain(b"/".into_iter())
			.chain(encoded_bn)
			.copied()
			.collect::<Vec<u8>>()
	})
}
```

![s1](/images/s1.jpg)

2. 使用 js sdk 从浏览器 frontend 获取到前面写入 Offchain Storage 的数据

![s1](/images/s1.jpg)

3. 链上随机数与链下随机数的区别：

  链上随机数是确定性的随机，随机种子是确定的但不可预测；
  链下随机数是不确定的随机，能获得真实现世界的随机性。
