# Debug log

## Bugs
### CC-01 : "The search suggestions are behind everything"

**Reproduced:** The issue with the matching search items hidden behind the rest of the page, typing anything does not show the correct results.

**Cause:** Most definitely a small enough z-index which causes the elements of the search bar to render behind the other sections which have a higher z-index.

**Fix:** Set the z-index in [style.css](./frontend/style.css#L758) to an arbitrarily high value so the search bar elements render in front always.

**Checked:** Search bar displays the matched results properly now.

**Time:** About 1 minute. Such issues are almost always correspond to lower z-index values.

---

### CC-02: "Can't read anything in dark mode"

**Reproduced:** The name and price of the dish are not visible in dark mode on the dish cards.

**Cause:** The issue was because of a hard coded hex which did not inherit the value from the parent element, causing the title and the prices to be rendered in a dark toned colour irrespective of the theme. This was identified by following the inheritance trace of the `color` property on the `name-btn` class:
![name-btn styles](./bug-ss/cc-02.png)

**Fix:** The issue was fixed by changing the `color` in [styles.css](./frontend/style.css#L295) property to inherit the parent elements style so the theme specific hex is respected.

**Checked:** The names and the price are properly displayed.

**Time:** About 2 minutes. Followed the style traces in dev tools.

---

### CC-03: "The menu is wider than my phone"

**Reproduced:** The issue is **unreproducible**. I tried all viewports of a standard phone device, the cards were responsible on all except below 305px (where the favourite/star button goes out of screen). Making the cards responsive for a smaller screen is unrealistic (and frankly unachievable today) for a modern smartphone since all of them have pretty much 360px+ width. The narrowest I tried is 320px with the star button just clipping out, although the functionality was still retained.

---

### CC-07: "Cancelling makes it worse"

**Reproduced:** The issue was reproduced by placing an order of 1 Rajma Chawal (with 5 in stock). Upon cancelling this order, the quantity was reduced to 3.

**Cause:** The issue was caused by decrementing the quantity of the ordered items from the current available stock (after order) in the [`releaseStock`](./frontend/js/validate.js#L123) function. For instance, the stock of Rajma Chawal was 5. When an order was placed for 1 Rajma Chawal, the quantity was decreased to 4. The `releaseStock` further decreased the incoming quantity (i.e the quantity of the item to be cancelled, which is 1 here) from the available stock after cancellation (i.e 4 here), resulting in 3.

The issue was identified by following the [`validation.releaseStock`](./backend/routes/orders.routes.js#L235) call to [`releaseStock`](./frontend/js/validate.js#L119).

**Fix:** The issue was fixed by simply changing the `-` sign to `+` so the quantity of the cancelled item is added back to the shelf, rather than removing.

**Time:** About 2 minutes to follow the trace and identify the issue.

---

### CC-06: "I ordered more than they had"

**Reproduced:** Ordering more than the available stock is possible.

**Cause:** This happens because the quantity of the dish in the cart is not validated against the stock of the item itself, allowing users to add as much of the dish as they want.

**Fix:** The issue was fixed by simple adding a new comparison to check whether the quantity of the stock is less than the quantity of the dish being added. If this is true, then an error is shown to the user.

**Time:** About 3 minutes.

---

### CC-08: "An old coupon still works"

**Reproduced:** The code FRESHERS24 is applicable on the checkout page.

**Cause:** The `GET /coupons` route at [coupons.routes.js](./backend/routes/coupons.routes.js#L22) loads the [coupons.json](./backend/data/coupons.json) file for checking the available coupons. Since the FRESHERS24 was still specified here, it was still usable.

**Fix:** The FRESHERS24 object from the coupons array was removed from the JSON file.

**Time:** About a minute.

---

### CC-09: "The menu shows more dishes than it should"

**Reproduced:** The menu page showed all 37 available dishes by default, contrary to the expected step loading behaviour.

**Cause:** The issue was caused because the [`paginate`](./backend/logic/search.js#L94) function returned the provided list of items as-is without any modifications.

**Fix:** The issue was fixed by simply returning the slicd list of items instead of the entire list as is.

**Checked:** The issue is now fixed with 5 dishes shown by default, adding 5 more on each scroll.

**Time:** About 5 minutes.

---

### CC-10: "Sorting by price is backwards"

**Reproduced:** The "low to high" and "high to low" filters on the Menu page return opposite results.

**Cause:** The issue was because of the wrong comparison functions being supplied to the sort function, as identified in [`search.js`](./backend/logic/search.js#L79). `a` denotes the first element, `b` denotes the second element the sort function considers. Since the comparison function is `a.price - b.price`, if the output of this is negative, a.price is placed before b.price, i.e the function will sort items in ascending order. The original definition however had the opposite order: `b.price - a.price` which caused items to be sorted in descending order in the "low to high" filter. Same with "high to low" filter.

**Fix:** The issue was fixed by simply swapping the comparison functions for `price-asc` and `price-desc` filters in [`search.js`](./backend/logic/search.js#L79).

**Time:** About 2 minutes

---

## Extras
## Unreported Bugs
### CC-U1: Price Filter on Menu Page shows Incosistent Results

**Reproduced:** The price slider/filter on the menu page showed incosistent results, such as showing all cards even though the lowest price filter (under 20 rupees) was selected, despite there being only one such applicable item. The issue was however not observed when the slider was slowly decreased to the lowest filter, the correct dish was then shown.

**Cause:** The issue was caused by the [`paginate`](./backend/logic/search.js#L94) function on the menu page returning all items as is, which produced stale data. 

**Fix:** The issue was fixed by returning the actual sliced items in the `paginate` function instead of the original list as is.

**Checked:** The issue has been fixed and the dish cards are correctly shown according to the price filter.

**Time:** About 15 minutes. I thought the issue was being caused by updates to the global `state` variable and I kept looking for all definitions which updated the `state` variable and finally stumbled upon a `paginate` function which controlled all the data being rendered on the page.

### CC-U2: Orders Page does not Show Previous Orders

**Reproduced:** The orders page showed no orders at all.

**Cause:** The issue was caused by a missing `userId` query parameter on the `GET /orders` route. The route logic at [`orders.route.js`](./backend/routes/orders.routes.js#L187) filters all orders so that only the orders belonging to a specific user are returned. Since `req.query.userId` was empty, an empty array was returned.

**Fix:** The issue was fixed by passing the current user's device ID in [api.js](./frontend/api.js#L155) in the userId param which returned all the orders properly.

**Time:** About 10 minutes spent to correlate how the actual requests were made to the API, i.e how the frontend communicated with the backend. The code is unnecessarily verbose :).

### CC-U3: Auto Refresh on Counter Page Does Not Persist

**Reproduced:** The auto refresh toggle on the Counter page did not persist through refreshes because its state was being saved no where. If someone turned off auto refresh and and refreshed the page, the toggle would have turned on again and the interval would have kept triggering.

**Cause:** The issue was caused because the state of the toggle was not being saved.

**Fix:** The fix was to simply save the state of the toggle (whether checked or unchecked) in local storage, and load that state when the counter page is rendered. Based on the value, the interval for refresh was created. Implementation can be found [here](./frontend/js/staff.js#L31).

**Time:** About 5 minutes.

## Notes
I have solved almost all the bugs. I also found a few unreported ones which are documented above.

The only unsolved bugs remaining are CC-04 and CC-05. I could not solve these due to a lack of CSS knowledge (especially responsiveness). I have prior experience with backend and server side dev only (my work can be found at github.com/FC5570 and github.com/ukaushik0712).

While checking CC-03, I looked for the slider issue mentioned in CC-05 aswell. I could not find this, but I am not jumping to a "unreproducible" conclusion just yet since I have not extensively looked for the issue. Due to personal time constraints, I could not devote much time to this specific task, otherwise solving CC-05 aswell could have been a possibility.

A few nitpicks:
1. Dish cards have ratings with no actual ratings. Upon further investigation I found the ratings default to somewhere around the average rating for the canteen (found in [`ratings.js`](./backend/logic/ratings.js))
2. Would be better UX if the menu prevented adding more than 30 dishes (the upper cap shown during checkout) while the order is being placed instead of at checkout.
3. There is no way to mark an order `picked` or `refunded`. 