# Code Refactoring for File Upload
---

| CIP | Title | Author | Status | Type | Category | Created |
| --- | --- | --- | --- | --- | --- | --- |
| 09 | Code refactoring for file upload | Su-nl-it | Review | Standards | Core | 2023/11/25 |

## Abstract

The current file upload process has become increasingly complex, leading to higher development costs in subsequent version iterations. Additionally, in order to reduce the workload for a transaction and minimize the use of loop statements, it is necessary to refactor the relevant code for the current file upload. Furthermore, a redesign of the metadata structure for storing file orders is required to minimize the space used for on-chain storage.

## Specification

### Core Changes

Refactor the code without affecting the upload process.

### Change Details

1. Code Architecture Refactoring

Restructured the code for transactions like upload_declaration, deal_timing_task, transfer_report, and calculate_end in file-bank. Separated them into functional invocation layers and storage change layers. Also, extracted some scattered code into methods or functions to improve code readability.

2. Metadata Storage Structure Adjustment

Reduced the space it occupies (anticipated a reduction by half).

Old one:
```
pub struct DealInfo<T: Config> {
	pub(super) stage: u8, 
	pub(super) count: u8,
	pub(super) file_size: u128,
	pub(super) segment_list: BoundedVec<SegmentList<T>, T::SegmentCount>,
	pub(super) user: UserBrief<T>,
	pub(super) miner_task_list: BoundedVec<MinerTaskList<T>, ConstU32<ASSIGN_MINER_IDEAL_QUANTITY>>,
	pub(super) complete_list: BoundedVec<u8, T::FragmentCount>,
}
```
New design:
```
// Remove the `miner_task_list` field and adjust the data type of the `complete_list` field.
pub struct DealInfo<T: Config> {
	pub(super) stage: u8, 
	pub(super) count: u8,
	pub(super) file_size: u128,
	pub(super) segment_list: BoundedVec<SegmentList<T>, T::SegmentCount>,
	pub(super) user: UserBrief<T>,
	pub(super) complete_list: BoundedVec<CompleteInfo<T>, T::FragmentCount>,
}

pub struct CompleteInfo<T: Config> {
	pub(super) index: u8,
	pub(super) miner: AccountOf<T>,
}
```
(_Due to the removal of the `miner_task_list` field mentioned above, data transmission no longer relies on reading this field but is now based on the allocation rules for transmission in DeOSS._)

### Modules requiring updates include

1. cess-node
  - Refactored code related to file uploads by breaking down functions into different processing layers, ensuring a clear and maintainable architecture for future updates.
  - Refactored file upload process code based on the new order metadata structure, reducing the use of loops to avoid excessive work in a single transaction.

2. deoss
  - Adjusted data transmission strategies and relevant code for order declarations based on the new order metadata structure.

3. cess-bucket
  - Adjusted storage node reporting interface code based on the new order metadata structure.

4. go-sdk
  - Adjusted the query metadata structure and corresponding transaction structure parameters.
### Data transmission rules:

DeOSS directly sends file data to the corresponding miner according to the allocation principle. The determination of which data to send depends on the completion status of the on-chain order metadata and the current transmission order of Deoss.

## Copyright
Copyright and related rights waived via [CC0](../LICENSE.md).
