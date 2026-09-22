---
description: A shortlist of Booking.com hotels for a destination, dates and party
---

Build a hotel shortlist.

Ask me for the destination, the dates and the party if I have not given them. Booking needs all of it: check-in, check-out, rooms, adults, and children with their ages.

Then:

1. Call `hasdata_booking_search_getBookingSearchResults` with those values, plus `price_max_` if I gave a budget. Set `currency` when I gave the budget in one, because the band is read in that currency.
2. Report how many properties came back. Nothing available means the dates are full, not that the destination is wrong.
3. Shortlist five: name, `pricePerStayParsed`, the `excludedChargesParsed` beside it when the result carries the pair, the star `rating` out of 5, the guest score `reviews.score` out of 10 with `reviews.count`, and the distance to the centre if the payload carries it. Label the two scores so nobody reads a 3 and a 9.4 as the same measure.
4. Sort by value rather than raw price, and say what you traded. The cheapest room in the wrong district is not the cheapest stay.
5. Open `hasdata_booking_place_getBookingPlaceDetails` for the two I pick, passing the same dates and party alongside the URL, because that tool requires them too. Report the room types, the cancellation terms and what recent reviewers mention.

Every number belongs to the dates I asked about. Repeat them next to the prices so nobody quotes a total for the wrong weekend.
