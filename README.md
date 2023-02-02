# Interview
## User Flow
* Show/search/find events
* User selects one
* If the eventId of the selected event has a lottery running, the `Enter Lottery` button is visible amongst other Information about the event. This is checked with the data returned from `/api/get_event` ajax call.  A large object with one of such keys being `runningLottery` - a lotteryId, 0 if not running a lottery.
* User clicks 'Enter Lottery' button.
* Page uses more screen space for visual content to give the user a strong feel for the event which they are thinking to attend. They're asked how many tickets they want, one or two.  Provides a **href** link to learn about the lottery in case they are too confused to move on. And has an **exit** button in the corner.
* User is asked to select showtime preferences, is able to select multiple. Visuals of the event content above (which still takes a large majority of screen space) have changed/zoomed to keep the user engaged compared to last scene. This time the exit button is a back arrow in case they want to change how many tickets were selected.
* Now after clicking **Next** a confirmation dialog is shown. X event, X tickets, for X showtimes, is the information correct? Confirm Enter Lottery?
* At this point the information cannot be changed and the user knows they'll enter lottery after clicking *Confirm Enter*.  The backend will be sent an ajax **fetch POST** `/api/lottery/enter` with the supplied details for entering lottery.
* The user will be told if they are in the lottery depending on the data returned from the POST request. They also are told where they can view the status of their tickets in the UI. Back to home button also supplied.  An option to send a push notification to their device upon winning is also shown, since we're assuming they are same day discounted tickets.
* Now the user has to wait until the lottery is drawn.  The more people who enter the lottery and less tickets in stock, both decrease the probability of winning the lottery. Could be good idea to have more **winners** than **tickets in stock** for the case when Users do not buy even though they won the lottery.
* User is notified of lottery win or discovers it via viewing Lottery tab.

## Terminology
`bookingID` = eventID with a specific showTime.  
There would be a bookings table in back-end db. Which also holds reference to a seats table.  
On successful booking the seats table would be marked as taken, from being vacant.

## Events
### on Select Show/Event in front-end
`/api/get_event` *takes:*  
* **eventID**

returns eventObject which contains non-empty `runningLottery` list, if event is running a lotto.

Used when showing `Enter Lotto` button or not.
### on Enter Lottery Event
`/api/lottery/enter`, *takes:*  
* **object(eventID)** 
* **object(lotteryID)**
* **number(num_tickets)** 
* **list(showtimes)** 

returns success code

Here the back-end will append to `db[userId]["pendingLottos"]` an object with {lotteryId, eventID, num_tickets, showtimes}.

### onPopulate pendingLottos list ui
* `/api/lottery/get_pending` *takes :*  
**userID**

returns list of awaiting pendingLotto objects.  
`{lotteryId,eventID,num_tickets,showtimes}`

### on Lotto Draw timer back-end
Lets assume that the backend appends to a list upon parsing all lottery participants at roll time.  

It would have had to pop from the db[userId]["pendingLottos"] list, and if the User gets lucky and wins as output from the winLotto algorithm, then lookup a valid bookingId to be placed in `db[userId]["discountedAccess"]`.

### on Purchase Tickets
`/api/lottery/get_discounted`, *takes*:  
*  **userID**

returns the **discountedAccess list** for userId. 

Thus giving the front-end access to `bookingID` and `lotteryID`.  
If any of the bookingID's match the currently viewed event/booking, then the user can apply the bonus thus reducing the cost of the ticket.  
Once the purchase is complete, the back-end removes the discountedAccess entry from list.

## Api's used
`api/get_event` - when selecting an event, populates the screen, reveals if lotto is active for said event.  
`api/lottery/enter`  - register a `pending lotto` for that user - essentially they are lotto win preferences.  
`api/lottery/get_pending` - get the list of pending lottos for user.  
`api/lottery/get_discounted` - get the list of discountedAccess for user. BookingId's for which a user is eligble discounted prices.  


## Pseudo-code.

```typescript
// Entering the lottery
function enterLottery(show: Show) {
  return (
    <form onSubmit={submitLotteryEntry}>
      <input type="text" name="name" placeholder="Name" />
      <input type="email" name="email" placeholder="Email" />
      <button type="submit">Submit</button>
    </form>
  );
}

// Submitting the lottery entry
async function submitLotteryEntry(event: FormEvent) {
  event.preventDefault();
  const formData = new FormData(event.target);
  try {
    const response = await fetch('/api/lotteries', {
      method: 'POST',
      body: formData
    });
    const data = await response.json();
    if (data.success) {
      // Show success message
    } else {
      // Show error message
    }
  } catch (error) {
    // Show error message
  }
}

// Purchasing the tickets
function purchaseTickets(show: Show) {
  return (
    <form onSubmit={submitTicketPurchase}>
      <input type="text" name="creditCard" placeholder="Credit Card" />
      <input type="text" name="expirationDate" placeholder="Expiration Date" />
      <input type="text" name="securityCode" placeholder="Security Code" />
      <button type="submit">Purchase</button>
    </form>
  );
}

// Submitting the ticket purchase
async function submitTicketPurchase(event: FormEvent) {
  event.preventDefault();
  const formData = new FormData(event.target);
  try {
    const response = await fetch('/api/tickets', {
      method: 'POST',
      body: formData
    });
    const data = await response.json();
    if (data.success) {
      // Show success message
    } else {
      // Show error message
    }
  } catch (error) {
    // Show error message
  }
}
```
