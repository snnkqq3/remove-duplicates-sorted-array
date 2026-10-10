# remove duplicates from sorted array: The Two-Pointer Solution for LeetCode 26 in Python, Java and C++, Plus Why `set()` Fails

You paste a solution that looks right, hit submit, and the judge says **Wrong Answer**. Usually it's the same three lines: you built a new list, or you used a set, or you returned the array instead of `k`. The problem itself takes about ten lines. The part people get wrong is understanding what the judge is actually checking.

This is LeetCode 26, the "easy" array question that shows up everywhere — phone screens, new-grad loops, and as a warm-up in almost every array-focused interview. Here's the full picture: the write/read pointer pattern that solves it, the trace on the canonical example, the edge cases that trip people up, and how it generalizes to the follow-up variants.

## What the problem actually asks for

Given `nums` sorted in non-decreasing order, remove duplicates in place so each unique value appears once, keep the relative order, and return `k`, the number of unique elements.

The confusing part is the wording about "the first k elements." Several people have asked about this on Stack Overflow after seeing output like `[1,2,_]`, assuming the leftover values need to be deleted somehow. They don't. The judge runs roughly this:

python
k = removeDuplicates(nums)
assert k == len(expectedNums)
for i in range(k):
    assert nums[i] == expectedNums[i]


Only the first `k` slots are read. Whatever garbage sits after index `k - 1` is invisible to the checker. You don't clean it, you don't shrink the array (you can't — arrays in Java and C++ have fixed size), and you don't return a new list. That single detail explains most of the failed submissions on this problem.

Two hard constraints do the rest of the work:

- **O(1) extra memory.** "Do not allocate extra space for another array."
- **O(n) time,** ideally — which you get for free from the sorted property.

## Why sorting is the whole trick

In an unsorted array, you'd need a hash set to detect duplicates, because two equal values can be anywhere. In a sorted array they're always **adjacent**. That means a duplicate is, by definition, an element equal to the one right before it.

So "remove duplicates" becomes "keep only the elements that differ from their predecessor" — a local comparison instead of a global lookup. No extra memory, one pass.

Worth remembering as a general interview note: the sorted precondition is load-bearing. Remove it and this approach collapses; you'd be back to sorting first (O(n log n)) or hashing.

## The write/read pointer solution

Two indices. `read` walks the whole array. `write` marks where the next unique value belongs. The invariant: at the start of each iteration, `nums[0..write-1]` already holds the distinct values seen so far, in order.

python
class Solution:
    def removeDuplicates(self, nums: list[int]) -> int:
        if not nums:
            return 0
        write = 1
        for read in range(1, len(nums)):
            if nums[read] != nums[write - 1]:
                nums[write] = nums[read]
                write += 1
        return write


java
class Solution {
    public int removeDuplicates(int[] nums) {
        if (nums.length == 0) return 0;
        int write = 1;
        for (int read = 1; read < nums.length; read++) {
            if (nums[read] != nums[write - 1]) {
                nums[write++] = nums[read];
            }
        }
        return write;
    }
}


cpp
class Solution {
public:
    int removeDuplicates(vector<int>& nums) {
        if (nums.empty()) return 0;
        int write = 1;
        for (int read = 1; read < nums.size(); ++read) {
            if (nums[read] != nums[write - 1]) {
                nums[write++] = nums[read];
            }
        }
        return write;
    }
};


Starting `write` at 1 (not 0) is the standard shortcut: in a non-empty array, the first element is always unique, so you can copy it in place conceptually and start scanning from index 1.

### Trace: `nums = [0,0,1,1,1,2,2,3,3,4]`

| read | `nums[read]` | `nums[write-1]` | Action | `write` after |
| --- | --- | --- | --- | --- |
| 1 | 0 | 0 | equal → skip | 1 |
| 2 | 1 | 0 | new → `nums[1] = 1` | 2 |
| 3 | 1 | 1 | skip | 2 |
| 4 | 1 | 1 | skip | 2 |
| 5 | 2 | 1 | new → `nums[2] = 2` | 3 |
| 6 | 2 | 2 | skip | 3 |
| 7 | 3 | 2 | new → `nums[3] = 3` | 4 |
| 8 | 3 | 3 | skip | 4 |
| 9 | 4 | 3 | new → `nums[4] = 4` | 5 |

Final array contents: `[0,1,2,3,4,2,2,3,3,4]`, return `5`. The trailing `2,2,3,3,4` are stale leftovers — exactly the thing the judge ignores.

### A second valid variant

Comparing the current element to its immediate predecessor works identically:

python
if nums[read] != nums[read - 1]:
    nums[write] = nums[read]
    write += 1


Both runs are O(n)/O(1), and both are accepted. The difference is conceptual: the first version compares against the last value you wrote, which is what makes it a genuine two-pointer pattern. The second compares within the input. If an interviewer asks you to explain the invariant, the write-pointer framing is easier to defend.

## Why `nums = set(nums)` fails

The most common wrong answer, in Python at least:

python
class Solution:
    def removeDuplicates(self, nums: List[int]) -> int:
        nums = set(nums)
        return len(nums)


Three separate problems, and it's worth being able to name all three:

1. **It isn't in place.** `nums = ...` rebinds the local name to a new object. The caller's list is untouched. Assigning to `nums[:]` would mutate the original; plain assignment does not. This distinction between mutating an object and rebinding a variable is what the Stack Overflow thread on this exact problem keeps coming back to.
2. **A set doesn't preserve order.** Same reason you'd use `dict.fromkeys(nums)` rather than `set(nums)` if you wanted an order-preserving one-liner.
3. **It wastes memory.** O(n) auxiliary space, which the problem explicitly forbids.

And even if you fixed the binding, you'd be running O(n log n) time (sorting the set back) for a task the sorted input already lets you do in linear time. The problem is essentially a test of whether you noticed that.

## Complexity, stated plainly

| Approach | Time | Space | In place? | Accepted? |
| --- | --- | --- | --- | --- |
| Set / sorted set | O(n log n) | O(n) | No | No |
| `dict.fromkeys(nums[:])` assignment | O(n) | O(n) | Yes (via slice assignment) | Yes, but not the intended answer |
| Write/read two pointers | O(n) | O(1) | Yes | Yes |

Note that the O(n) + O(n) slice variant is technically accepted by the judge while violating the spirit of the problem. In an interview, that's the version that gets you the follow-up question.

## Edge cases worth checking before you submit

- **Empty array.** LeetCode's current constraints say `1 <= nums.length`, but the guard costs one line and interviewers sometimes relax the constraint. Return 0.
- **Single element.** `write` stays at 1, loop body never runs, return 1. Correct.
- **All elements identical,** e.g. `[5,5,5,5]`. Every comparison is equal, nothing is written, return 1. `nums[0]` is the single unique value.
- **Negative values.** Irrelevant to the logic. `-100` as a constraint bound changes nothing.
- **Initializing `write = 0`.** This is the most common off-by-one. With `write = 0` you'd compare `nums[read]` against `nums[-1]` in Python (the last element, silently wrong) or index out of bounds in Java/C++.

## The follow-up: duplicates allowed at most twice

LeetCode 80 extends the same problem: each unique element may appear at most twice. The two-pointer skeleton survives with a single change — start `write = 2` and compare against `nums[write - 2]`:

python
if nums[read] != nums[write - 2]:
    nums[write] = nums[read]
    write += 1


Why `write - 2` works: after you've written elements correctly, `nums[write - 2]` is the second-to-last value you kept. If the incoming value equals it, then the last two written values are both that value, and adding a third would break the "at most twice" rule. Generalizing to "at most `m` occurrences" gives `nums[write - m]`.

Being able to make that jump is the real payoff of learning the pattern rather than memorizing the solution.

## Where this pattern shows up next

The write/read pointer — sometimes called the slow/fast pointer within a single array, or "in-place compression" — is one of a small set of reusable array templates:

- **27. Remove Element** — same skeleton, compare against a target value instead.
- **283. Move Zeroes** — write pointer skips zeros, then pad the tail.
- **26 / 80. Remove Duplicates I and II** — what we just did.
- **75. Sort Colors** — a three-way variant, where the write pointer logic gets a "middle" pointer.

The interview skill isn't recognizing problem 26 when you see it again. It's hearing "in place, O(1) space, sorted input" and immediately suspecting a read/write pointer scheme.

If you want that mapping spelled out — problem shape to pattern, with code templates in several languages — that's the actual product AlgoMonster sells. Its Two Pointers module is built around exactly this class of problem, and the platform frames each lesson as "which pattern does this belong to and why," rather than dropping you into another solution write-up. Worth a look only if pattern recognition is the thing tripping you up, not if you're just here for LeetCode 26.

## AlgoMonster plans and pricing

AlgoMonster has no monthly tier. You're choosing between a free tier, an annual Pro plan, and a one-time lifetime purchase. Current figures from the subscribe page:

| Plan | What you get | Price | Billing | Link |
| --- | --- | --- | --- | --- |
| Free | Limited sample access to lessons and problems | $0 | — | [Try the free tier](https://algo.monster?fpr=li69) |
| Pro — Yearly | All patterns, lessons and problems; unlimited AI assistant; unlimited Speedrun; real company questions updated monthly; 500+ lessons across 6 courses, 1,000+ illustrations, 3,000+ editorials; member discount on live bootcamps | $99/year (about $8.25/month, roughly $0.28/day) | Billed annually, 12 months of access | [Get Pro Yearly](https://algo.monster?fpr=li69) |
| Pro — Lifetime | Everything in the annual plan plus lifetime updates | $189 | One-time payment | [Get Pro Lifetime](https://algo.monster?fpr=li69) |

Two extras come bundled at no additional cost: a Foundation Course covering data structures and algorithms from the ground up, and a "Build an LLM" course that goes through tokenization, attention, and training rather than stopping at API usage. Courses developed with external partners, such as Quant Interview Prep, are sold separately.

The honest read on the math: at $189 one-time, lifetime pays for itself after two annual renewals. If you're prepping for a single job search over the next two or three months and you already know what to study, the free tier plus a problem list like the one above is enough. The paid plans make sense if you want the structured pattern library, the templates, and the decision flowcharts you'd otherwise have to assemble from scattered write-ups. There's no team or enterprise option — it's built for individual candidates.

## Quick answers

**Does the leftover part of the array matter?** No. Only indices `0` to `k-1` are checked.

**Can I use a set?** Only if you write it back into the original array, and you'll burn O(n) space plus O(n log n) time doing it.

**Do I need to handle an empty array?** LeetCode now guarantees at least one element. Add the guard anyway — it costs nothing and the constraint is easy to get asked about.

**Is comparing to `nums[read - 1]` wrong?** No. It's a valid equivalent variant, also O(n)/O(1).

**What if the array isn't sorted?** Then this pattern doesn't apply. Sort it first if you're allowed to, or switch to hashing.
