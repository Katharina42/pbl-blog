---
layout: post
title: "Guardian of the Groceries: Navigating through Time and Space of my Spring Boot Project"
---
Hello! My name is Katharina Lechner and I've recently completed my 10-month programming journey at [everyone codes](https://everyonecodes.io/). Having acquired proficiency in Java and Spring Boot, I was ready for a new challenge: a 6-week project where I could apply my newfound knowledge and craft something of my own.
Like everything in the universe, our food is also subject to the passage of time. That's why I decided to create an app that allows you to track perishable goods along with their respective expiration dates...and I named it.. 
# Guardian of the Groceries

## The Guardian's Requirements
To shed light on the Guardian's vision, the app's requirements unfold as follows:

* The Guardian ensures that users possess the power to add, remove, and modify distinct storage locations such as "refrigerator" or "freezer."
* Empowered by the Guardian's guidance, users gain the ability to manage items effortlessly—adding, removing, or updating them, with the caveat that each item finds its dwelling within a solitary storage location.
* Within the sacred confines of each storage location, the Guardian grants users a glimpse into an organized list of items. The ones destined for consumption or use take precedence, gracefully ascending to the top of the list.
* Ever watchful of the temporal tapestry, the Guardian orchestrates a display of the time remaining until the expiration date of each item, skillfully presenting it in a relatable format— be it "in 21 days" or "21 days ago."

## The Guardian's dilemma 

In the intricate journey of crafting this project, there came a defining moment when the Guardian, the vigilant overseer, orchestrated the creation of two vital entities: StorageLocation and Item.

### The Storage Location
```
@Entity

public class StorageLocation {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String locationName;

    @JsonBackReference
    @OneToMany(mappedBy = "storageLocation")
    private List<Item> items;
}
```
### The Item
```
@Entity

public class Item {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    private LocalDate expirationDate;

    private Period period;

    @ManyToOne
    @JoinColumn(nullable = false, name = "storage_location_id")
    private StorageLocation storageLocation;

    public void setPeriod() {
        if (this.expirationDate == null) {
            this.period = null;
        } else this.period = Period.between(LocalDate.now(), this.expirationDate);
    }

    public String getPeriodInReadableFormat() {
        if (this.period == null) {
            return null; 
        } else {
            String periodString = "";
            if (this.period.equals(Period.between(this.expirationDate, LocalDate.now()))) {
                return "today";
            }
            if (this.period.getYears() != 0) {
                periodString += this.period.getYears() + " years ";
            }
            if (this.period.getMonths() != 0) {
                periodString += this.period.getMonths() + " months ";
            }
            if (this.period.getDays() != 0) {
                periodString += this.period.getDays() + " days";
            }
            periodString = periodString.trim().replace('-', ' ');
            if(this.period.isNegative()) {
                return  periodString + " ago";
            } else {
                return "in " + periodString;
            }
        }
    }
}
```

At this juncture, the Guardian unveils the ethereal forms of StorageLocation and Item entities, each Spring Boot layer contributing to their existence, awaited the user's interaction, promising a harmonious dance of data and functionality orchestrated by the vigilant Guardian.

But...there was a problem...

...a disturbance lingered within the serene dance. Each time the user cast a gaze upon the storage locations, a cascade of redundant computations unfurled—an incessant recalculation of expiration periods for every item.
The threads of the Spring Boot layers, while robust, struggled under the weight of this computational burden.