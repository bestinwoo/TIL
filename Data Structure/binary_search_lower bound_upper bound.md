이진 탐색 Lower Bound, Upper Bound
==================================

이진 탐색은 찾는 값이 존재하는지 탐색하여 하나의 값을 찾는 것이라면 lower bound와 upper bound는 이진 탐색에서 살짝 변형시킨 방식으로 **중복된 값이나 상한, 하한선을 찾는 경우** 등의 다양한 방식으로 활용할 수 있다.
[이진탐색 포스팅](https://bestinu.tistory.com/7)

## Lower Bound
Lower Bound는 타겟보다 같거나 큰 값이 나오는 처음 위치를 찾는다.
[1, 2, 3, 3, 3, 4, 5] 라는 리스트에서 3을 타겟으로 Lower Bound를 찾으면 결과로 나오는 인덱스는 3이 처음 등장하는 2가 된다.

## Upper Bound
Upper Bound는 타겟보다 큰 값이 나오는 처음 위치를 찾는다.
위와 같은 [1, 2, 3, 3, 3, 4, 5] 의 리스트에서 3을 타겟으로 Upper Bound를 찾으면 결과로 나오는
인덱스는 3보다 큰 값인 4가 있는 위치인 5가 된다.

**Lower Bound**
```C++
int lowerBound(int target) {
	int low = 0;
	int high = list.size();
	while(low < high) {
		int mid = (low + high) / 2;
		if(target <= list[mid]) {
		// target이 mid와 같은 경우에도 high만 변하므로 lowerBound 된다.
			high = mid;
		} else {
			low = mid + 1;
		}
	}
	return low
}
```
**Upper Bound**
```C++
	int upperBound(int target) {
	int low = 0;
	int high = list.size();
	while(low < high) {
		int mid = (low + high) / 2;
		if(target < list[mid]) {
			high = mid;
		} else {
			low = mid + 1;
		}
	}
	return low
}
```




