# Two Sum Python: Brute Force vs. One-Pass Hash Map, Why [3,3] Breaks Your Code, and the Pattern Interviewers Want to Hear

Search "two sum python" and you'll get the same fifteen lines of code a hundred times over. Which is fine, except that copying those fifteen lines is not what makes you pass a screen. What makes you pass is knowing why the dictionary works, why the obvious one-liner fails on `[3, 3]`, and what to say when the interviewer asks you to do it without extra space.

So this is the version of the answer that includes the parts everyone skips: the edge cases, the complexity tradeoffs, the follow-up variants, and where a hash-map pattern like this one fits into a larger prep plan.

## What the problem is actually asking

The statement, in its standard form:

Given an array of integers `nums` and an integer `target`, return the indices of the two numbers that add up to `target`. Each input has exactly one solution, and you may not use the same element twice.

Three things in that sentence matter more than the code:

- **Indices, not values.** `[2, 7]` is wrong. `[0, 1]` is right.
- **Exactly one valid answer exists.** No need to handle "which pair if there are several" unless the interviewer changes the rules mid-problem, which they sometimes do.
- **You may not use the same element twice.** This is the clause that trips up most Python solutions built on `.index()`.

The official constraints put the array size at up to 10,000 elements, with values anywhere from −10⁹ to 10⁹. That ceiling is the reason the quadratic solution is a starting point and not an answer.

| Input | Target | Expected output |
| --- | --- | --- |
| `[2, 7, 11, 15]` | `9` | `[0, 1]` |
| `[3, 2, 4]` | `6` | `[1, 2]` |
| `[3, 3]` | `6` | `[0, 1]` |

That third row is the whole lesson compressed into two numbers.

## The O(n²) version, and why it's not a waste of time

python
class Solution:
    def twoSum(self, nums, target):
        for i in range(len(nums)):
            for j in range(i + 1, len(nums)):
                if nums[i] + nums[j] == target:
                    return [i, j]


Nested loops over every pair. Time complexity O(n²), space complexity O(1). For 10,000 elements you're looking at roughly 50 million comparisons in the worst case. A benchmark recorded in a public solutions repo clocked this brute-force version at around 3,653 ms, against about 51 ms for the one-pass dictionary approach. Same problem, same language, roughly seventy times the runtime.

Write it anyway. Every interviewer expects you to say the brute-force version out loud before optimizing, and the reason is not politeness. Stating the naive approach defines the problem, and it's from the naive approach that you notice the redundancy: for each element you're scanning the whole rest of the array looking for one specific value. Once you see that, the optimization writes itself, because "find one specific value in a collection" is exactly what a dictionary does in constant time.

## Two-pass dictionary: correct, mildly wasteful

The first improvement builds a full value-to-index map, then queries it:

python
class Solution:
    def twoSum(self, nums, target):
        index_of = {}
        for i, num in enumerate(nums):
            index_of[num] = i

        for i, num in enumerate(nums):
            complement = target - num
            if complement in index_of and index_of[complement] != i:
                return [i, index_of[complement]]


The `index_of[complement] != i` guard is doing real work here. Without it, `[3, 3]` with target `6` can return index `0` paired with itself. The same repo benchmark put this version around 70 ms.

It's a perfectly good solution. It's just one pass too many, and the guard is a bandage over a design choice rather than a property of the algorithm.

## One-pass dictionary: the version to write

python
class Solution:
    def twoSum(self, nums, target):
        seen = {}
        for i, num in enumerate(nums):
            complement = target - num
            if complement in seen:
                return [seen[complement], i]
            seen[num] = i


Read it as a sentence: as you walk the array, you keep a notebook of every number you've already passed and where it was. Before writing the current number down, you check whether its missing partner is already in the notebook. The first time that check succeeds, you're done.

The self-matching bug disappears on its own. You look up the complement *before* inserting the current number, so the current index can never be its own partner.

### Walking through [2, 7, 11, 15] with target 9

- `i=0`, `num=2`. Complement is `7`. Not in `seen` (empty). Store `2 -> 0`. `seen = {2: 0}`
- `i=1`, `num=7`. Complement is `2`. It's in `seen` at index `0`. Return `[0, 1]`

Two iterations out of four. The walkthrough matters more than it looks: in an interview, narrating those two lines is what demonstrates you understand the mechanism, not just the syntax.

### A case where the answer arrives late: [3, 2, 4] with target 6

- `i=0`, `num=3`. Complement `3`. Not in `seen`. Store `{3: 0}`
- `i=1`, `num=2`. Complement `4`. Not in `seen`. Store `{3: 0, 2: 1}`
- `i=2`, `num=4`. Complement `2`. Found at index `1`. Return `[1, 2]`

Notice that checking for `3` at the first step fails correctly, because `3 + 3 = 6` would require using index `0` twice.

## The edge cases that break naive Python solutions

A widely-shared Stack Overflow answer to this problem used `nums.index(j)` to find the partner's position, and it passed `[2, 7, 11, 15]` while failing `[3, 3]`. That failure pattern is worth understanding, because it shows up in variations every few months.

**Duplicate values.** `list.index()` returns the *first* occurrence of a value. With `[3, 3]`, both lookups point at index `0`, so the code either returns `[0, 0]` or rejects itself with a self-comparison check. A dictionary keyed by value has the same collision problem unless you either do the check-before-store ordering above, or store lists of indices per value.

**Negative numbers and zero.** These are not special cases for the hash-map approach, and it's worth saying so out loud. Subtraction handles negatives, and `0` is a valid complement (`target = 5`, `nums = [0, 5]`). Solutions that use a "seen" array or index arithmetic instead of a dict do break here.

**Arrays of length 2.** The minimum permitted input. `[3, 3]` and `[1, 2]` with target 3 are the two cases to mentally run.

**The guarantee itself.** The problem promises exactly one solution, so returning `None` at the end is defensive style rather than a requirement. If the interviewer removes that guarantee, you need a different return contract, which is usually where the follow-up conversation starts.

## Complexity at a glance

| Approach | Time | Space | Notes |
| --- | --- | --- | --- |
| Brute force, nested loops | O(n²) | O(1) | The baseline; ~3,600 ms in one repo's 10k-element-scale benchmark |
| Two-pass dictionary | O(n) | O(n) | Needs an index guard for duplicate values |
| One-pass dictionary | O(n) | O(n) | Standard interview answer; no guard needed |
| Sort + two pointers | O(n log n) | O(1) extra | Destroys original indices, so it only works when indices aren't required |
| Two pointers on sorted input | O(n) | O(1) | The intended answer for Two Sum II specifically |

The honest takeaway: you cannot beat O(n) time here, and getting O(1) space means either the input is already sorted or you're giving up index information. That tradeoff is a sentence interviewers like hearing.

## When the pattern changes

The same problem with one constraint removed becomes a different problem, and knowing which constraint changed is the actual skill being tested.

**Two Sum II, input array is sorted.** Now two pointers beat the hash map, because the sorted order gives you information the dictionary was spending memory to reconstruct. Move `left` right when the current sum is too small, `right` left when it's too large. O(n) time, O(1) space. If you're solving this one with a dictionary, you're answering a question nobody asked.

**Three Sum.** Sort, fix one element, then run the two-pointer scan on the remainder. The dictionary approach gets awkward because you now need to deduplicate triplets, and sorting gives you that for free.

**"Does a pair exist?" without indices.** A set is enough. No index tracking, no `enumerate`, fewer lines.

**Follow-up questions to expect.** "What if the array is a stream?" pushes you toward the one-pass solution and a note that you can't sort. "What if you want all pairs?" changes the return type and forces deduplication. "What if the input doesn't fit in memory?" is where the answer becomes about hash maps, load factor, and knowing your data.

## What to say out loud

Most candidates can produce the code. The difference in the room is narration, roughly in this order:

1. Restate the problem in one sentence, including the "one solution, no reuse" guarantees.
2. Offer the brute force and its complexity, then name the redundant work.
3. Propose the hash map, and explain the complement idea in plain terms.
4. Write the one-pass version, talking through the check-before-store ordering.
5. Run it by hand on `[2, 7, 11, 15]` and mention `[3, 3]`.
6. State O(n) time and O(n) space, and note that sorting would trade time for space if indices weren't required.

That's maybe four minutes. It's a much better four minutes than silently typing a correct answer.

## Practicing this as a pattern, not as one problem

Here's the thing about Two Sum: it isn't asked because interviewers care about pairs of integers. It's asked because it's the clearest possible test of whether you reach for a hash map when you see "for each element, look up something specific." That's the pattern behind subarray sum problems, group anagrams, LRU caches, and half the array questions in a FAANG loop.

If your current prep is grinding problems one at a time, the pattern framing is what's missing, and it's the gap AlgoMonster is built around. It's a text-and-diagram course put together by ex-Google engineers that organizes content around explicit coding patterns with reusable templates, rather than a problem bank you work through in arbitrary order. Two Sum itself lives in its hash map material, with the complement check-before-store ordering spelled out step by step and an animated walkthrough of the array as the dictionary fills up.

A few details that matter practically:

- **No video.** Everything is text plus roughly 678 interactive diagrams and 3,000+ editorials, which makes it fast to skim and easy to search. If you learn from watching someone else type, this format will frustrate you.
- **Templates in six languages.** Python, Java, JavaScript, C++, Go, and Haskell, so the Python template is the one you'll actually practice with.
- **Speedrun mode** for timed review passes, which is the closest thing in the product to interview pressure.
- **Bonus courses bundled in.** A Foundation Course for CS fundamentals and a "Build an LLM Course" covering tokenization, attention, and training. Both are included with a subscription rather than sold separately.
- **What it doesn't cover.** Reviews consistently flag the same three gaps: system design coverage that's introductory rather than sufficient for senior roles, minimal behavioral prep, and no human mock interviews. If your loop includes a deep system design round, budget for a second resource.

You can kick the tires without paying anything first: 👉 [**start on the free tier and work through the hash map lessons before deciding**](https://algo.monster?fpr=li69).

## Plans and current pricing

AlgoMonster runs one structure with three tiers, and no monthly option. The paid prices below reflect the promotional pricing currently shown on the official plans page; the strike-through figures are the listed full prices. Promotions change, so confirm what's live at checkout.

| Plan | What you get | Price | Billing | Purchase |
| --- | --- | --- | --- | --- |
| Free | Limited access to a subset of patterns and problems; enough to test the text-and-diagram format | $0 | None | [Explore the free content](https://algo.monster?fpr=li69) |
| Pro-Yearly | All patterns, lessons, and problems; unlimited AI assistant and Speedrun; 325 lessons and problems, 678 illustrations, 3,000+ editorials; company question banks updated monthly; member discount on live bootcamps; 3-day free trial on the annual term | $99/year (listed at $198), about $8.25/month billed annually | Annual, cancel anytime | [See the current annual price](https://algo.monster?fpr=li69) |
| Pro-Lifetime | Everything in the annual plan plus lifetime updates and an internal referral to a top tech company for qualified candidates | $189 one-time (listed at $378) | One payment, yours permanently | [Check the lifetime plan on the official page](https://algo.monster?fpr=li69) |

Quick math on the two paid tiers, since that's the actual decision. If you're prepping for one interview cycle in the next couple of months, the annual plan is the cheaper commitment and it's the one that matches a 4-to-8-week study window. The lifetime plan pays off if you expect to interview again in a few years, which is common enough in this industry that the difference isn't hard to justify. Note that third-party reviews quote a wide range of historical price points, from roughly $89 to $459, because the platform runs sales frequently. Treat any number you see quoted on a blog, including the ones above, as something to verify on the live page before you buy: 👉 [**compare the current Pro plans and active discount**](https://algo.monster?fpr=li69).

## Common questions

**Is the free tier enough to decide?** Yes for format. No for prep. Multiple reviews say the same thing: use it to find out whether you can learn from dense text and interactive diagrams, then commit or walk away.

**Does the lifetime plan include future content?** According to the plan description, yes: lifetime updates plus everything in the annual plan. Two Sum being a stable problem, the value of that clause is more about the new patterns and courses they add over time.

**Is a course necessary for a problem this well documented?** For Two Sum alone, no. You can read this page and be done. The reason to pay is the fifty problems that come after it, where the question shifts from "how do I solve this" to "which pattern is this," and that second question is what a curated curriculum actually teaches.

**Does it replace LeetCode?** No. The consistent advice across reviews is to learn the patterns on AlgoMonster and build volume on LeetCode. Roughly 325 lessons and problems will teach you the frameworks; they won't build the raw speed that a few hundred more solves will.

## The short version

The one-pass dictionary solution is the answer. Understand it as "for each number, check for its partner before recording it," not as a memorized loop, and the `[3, 3]` case takes care of itself. Know that you can't do better than O(n) time, know that sorted input flips you to two pointers, and be able to explain the tradeoff in two sentences.

Then practice the next twenty problems that use the same pattern, because the interview question was never about two numbers adding up.
