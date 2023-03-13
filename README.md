# System Design

## Parking Lot System Design
https://www.youtube.com/watch?v=eV5Xh6jNfmU&t=1s

Design Parking lot
- system design
- scale
- APIs
- Concurrency
- Distributed

### Use cases
- give user ticket when he enters
- generate price when user exits

### Details about the problem
- what detail should be in the ticket
  - vehicle number, time of entry, parking spot, unique id for tracking
- Allocatoin of parking to user
  - are there different types of parking?
    - small, medium, large
  - How is parking spot decided?
- What is the logic of generating price?

### Scale
- number of parking slots in one parking lot
  - floor in parking
  - parking slots per floor
  - 10 * 500 = 5k
- Data size for parking slot
  - number of parking slots * size per parking slot
  - 5k * 1k = 5M
- Data size for ticketing per day
  - (number of parking) * (size per ticket)
  - 10k * 100 = 1 MB
- Data for 10 years = 10 * 365 * 1MB = 3.65 GB (one machine)
- 1 million parking lots => distributed solution

### Public APIs
```java
interface ParkingService {
    Ticket entry(VehicleType type);
    double exit(long ticketId);
}

interface Ticket {
    long id;
    Date entryTime;
    Date exitTime;
    ParkingSlot slot;
    String vehicleNumber;
}

interface ParkingSlot {
    long id;
    VehicleType type;
    State state;
    // other metadata about floor layout etc.
}

enum VehicleType { SMALL, MEDIUM, LARGE }
enum State { OCCUPIED, UNOCCUPIED, UNDER_REPAIR }
```

### System design
When receive request update slot

![image](https://user-images.githubusercontent.com/27693622/224696895-d6e93801-68b4-40c9-887f-6bb78da6065b.png)

We also include a pricing service for pricing policies:

![image](https://user-images.githubusercontent.com/27693622/224697763-062ba590-bf96-4623-bbbe-3541cabcc7df.png)


### Internal APIs
```java
interface SlotService {
    ParkingSlot allocate(VehicleType type);
    boolean unallocate(long ParkingSlotId);
}

interface PricingService {
    double calculate(VehicleType type, Date inTime, Date outTime);
}
```

### Concurrency (more than one entry / exit)
- only need to solve for Slot Service
- Price service not affected
- Tickets are generated for one user
- Steps:
  - Query to find available slots
  - Pick slot to allocate
  - Try lock on slot. If unavailable wait. If fails pick another slot
  - Assign slot to vehicle make it occupied
  - unlock (application / db layer)
  - Better to take lock on DB layer

### Distributed (Large number of parking lots)
- Add parking lot id in Tickets, and ParkingSlot
- Shard the data
- Tickets and parking slot
  - shard on basis of parking lot id

### Conclusion
- good question for high and low level design
- big scope for question



