### DiffUtil
- 개념 및 정의
  + RecyclerView에서 데이터가 변경될 때 변경된 부분만 효율적으로 업데이트하기 위한 유틸리티 클래스.
  + 전체 리사이클러뷰를 다시 그리는 notifyDataSetChanged() 대신, Diff 알고리즘을 통해 바뀐 아이템만 갱신 → 성능 최적화 + UX 개선

- 기본 사용법
  + ```kotlin
    class MyDiffCallback(
      private val oldList: List<User>,
      private val newList: List<User>
    ) : DiffUtil.Callback() {
    
      override fun getOldListSize() = oldList.size
    
      override fun getNewListSize() = newList.size
    
      // 아이템 자체가 같은지 (고유 ID 비교)
      override fun areItemsTheSame(oldItemPosition: Int, newItemPosition: Int): Boolean {
        return oldList[oldItemPosition].id == newList[newItemPosition].id
      }

      // 아이템의 내용이 같은지 (equals 비교)
      override fun areContentsTheSame(oldItemPosition: Int, newItemPosition: Int): Boolean {
        return oldList[oldItemPosition] == newList[newItemPosition]
      }
    }

    // 사용
    val diffResult = DiffUtil.calculateDiff(MyDiffCallback(oldList, newList))
    adapter.submitList(newList)  // ListAdapter면 이거만 호출하면 됨
    diffResult.dispatchUpdatesTo(adapter)
    ``` 

- 📌 ListAdapter와 DiffUtil
  + 안드로이드 Jetpack에서 제공하는 ListAdapter는 내부적으로 DiffUtil을 내장하고 있음
  + 따라서 DiffUtil.ItemCallback<T>만 구현하면 자동으로 DiffUtil 적용 가능 
  + ```kotlin
    data class User(val id: Int, val name: String)

    class UserAdapter : ListAdapter<User, UserAdapter.ViewHolder>(DIFF_CALLBACK) {
        
        companion object {
            private val DIFF_CALLBACK = object : DiffUtil.ItemCallback<User>() {
                override fun areItemsTheSame(oldItem: User, newItem: User) = oldItem.id == newItem.id
                override fun areContentsTheSame(oldItem: User, newItem: User) = oldItem == newItem
            }
        }

        class ViewHolder(val binding: ItemUserBinding) : RecyclerView.ViewHolder(binding.root)

        override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
            val binding = ItemUserBinding.inflate(LayoutInflater.from(parent.context), parent, false)
            return ViewHolder(binding)
        }

        override fun onBindViewHolder(holder: ViewHolder, position: Int) {
            val user = getItem(position)
            holder.binding.textView.text = user.name
        }
    }
    ```

- DiffUtil vs notifyDataSetChanged
  + | 비교 항목      | notifyDataSetChanged() | DiffUtil           |
    | ---------- | ---------------------- | ------------------ |
    | 갱신 범위      | 전체 아이템 다시 그림           | 변경된 항목만 그림         |
    | 성능         | ❌ 비효율적                 | ✅ 효율적              |
    | UX (애니메이션) | ❌ 깜빡임, 스크롤 튐           | ✅ 자연스러운 애니메이션      |
    | 사용 편의성     | 간단하지만 성능 나쁨            | 초기 구현 필요하지만 재사용 가능 |
 

- 면접 예상 질문
  + DiffUtil을 직접 구현했을 때와 ListAdapter의 차이는?
    * DiffUtil은 직접 old/new 리스트를 비교하고 dispatchUpdatesTo() 호출 필요
    * ListAdapter는 DiffUtil을 내부적으로 관리하므로 submitList()만 호출하면 됨
  + DiffUtil의 내부 동작 원리 아나요?
    * 최소 편집 거리 알고리즘 (Myers’s diff algorithm) 기반으로 리스트 변경점 계산
    * O(n + m) 성능 (n = oldList, m = newList 크기)
  + DiffUtil에서 areItemsTheSame과 areContentsTheSame 차이는?
    * areItemsTheSame: 같은 객체(동일 ID)인지 확인 (ex: PK 비교)
    * areContentsTheSame: 같은 객체라도 내용이 바뀌었는지 확인 (equals 비교)


---
