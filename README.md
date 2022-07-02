# fix_float

Default float types in rust doesn't implement the standard traits Ord and Hash, which is theoretically correct but very annoying. This crate tries to handle some restriction on float values in order to finally implement these traits.

Because we are talking about floating numbers, comparing and hashing are subtle. The first attempt is to create a wrapper around floating numbers named ff64 and ff32, that prohibites edge values such as Nan (Not A Number) and +/- Infinite. This restriction is quite good because we would like to handle these edge cases seperataly and then concentrate on the 'happy path'.

## Demo

```rust
use fix_float::*;
use rand;

fn main() {
	let mut v: Vec<ff64> = vec![];

	for _ in 0..10 {
		v.push(ff64!(rand::random::<f64>()));
	}

	println!("values:");
	for &elem in &v {
		println!("  {:.2}", *elem);
	}

	v.sort();

	println!("values sorted:");
	for &elem in &v {
		println!("  {:.2}", *elem);
	}
}
```

## Examples

```rust
use fix_float::*;

// inputs: A random list of float numbers
// outputs: A tuple of three elements:
//    .0: sorted list of fix floats numbers
//    .1: numbers of NAN
//    .2: numbers of INFINITY (positive or negative)
fn double_triage(v: Vec<f64>) -> (Vec<ff64>, usize, usize) {
	let mut vff64 = Vec::new();
	let mut nb_nan = 0;
	let mut nb_infinity = 0;

	for &elem in &v {
		match ff64::try_from(elem) {
			Ok(x) => vff64.push(x),
			Err(ErrorTryFrom::CannotFixNan) => nb_nan += 1,
			Err(ErrorTryFrom::CannotFixInfinity) => nb_infinity += 1,
		}
	}

	vff64.sort();

	(vff64, nb_nan, nb_infinity)
}

#
```

This should work fine!
```rust
let a = ff64!(42.42);
let b = ff64!(0f64);
```

The three following snippets should panic!
```rust
let a = ff64!(f64::NAN);
```

```rust
let a = ff64!(f64::INFINITY);
```

```rust
let a = ff64!(f64::NEG_INFINITY);
```

## Useful Datastructures

A direct consequence is that we can now use fix floats in useful datastructures such as HashMap, HashSet, BinaryHeap. For instance, this is very useful when it comes to graph algorithms.

```rust
let map: HashMap<ff64, ()> = HashMap::new();
let set: HashSet<ff64> = HashSet::new();
let heap: BinaryHeap<ff64> = BinaryHeap::new();
```
