---
layout: post
title: "Navigating through Time and Space of my Spring Boot Project"
---
Hi, my name is Katharina Lechner and in order to fulfill a 10-month java programming course at everyone codes I had 6 weeks to create a small spring boot application of my choice. 
In this application the user is able to manage  and track food items along with their expiration date, that are stored in different storage locations. the item, as well as the storage location can be Created, Read, Updated and Deleted by the user.
to make the app more user-friendly I wanted to show the time until the item is expired in a relative way, such as 'in 21 days' or '21 days ago'.
## My Problem
At some point, while trying to implement the relative representation of time my item entity looked as follows:

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


As you can see storing dynamic properties in a database just for displaying on an HTML is not the most efficient approach, because it could impact the performance, especially if the properties are frequently accessed. With the getPeriodInReadableFormat function I also limited myself to an output in only one language.

## My Solution

After extensive research on time and dates in Java, I opted for a solution involving a Data Transfer Object (DTO) named ´ItemDTO´ and the PrettyTime formatting library.

I first incorporated PrettyTime into my project by adding the dependency to the Maven pom.xml:
```
<dependency>
   <groupId>org.ocpsoft.prettytime</groupId>
   <artifactId>prettytime</artifactId>
   <version>5.0.7.Final</version>
</dependency>
``` 
Next, I created the ItemDTO class:
```
public class ItemDTO {
    private Long id;
    private String name;
    private LocalDate expirationDate;
    private LocalDate wastedDate;
    private StorageLocation storageLocation;
    private String timeDiff;
    }
```
For computing the time difference, I introduced the setTimeDiff method:
```
    public void setTimeDiff() {

        PrettyTime prettyTime = new PrettyTime(Locale.forLanguageTag("en"));
        prettyTime.removeUnit(Hour.class);
        prettyTime.removeUnit(Minute.class);
        prettyTime.removeUnit(Second.class);
        prettyTime.removeUnit(Millisecond.class);
        (expirationDate);
        timeDiff = prettyTime.format(prettyTime.calculatePreciseDuration);
    }
```
With prettyTime you can choose the language and remove different time Units- as for my project I only wanted years, month and days I disabled all the units from milliseconds to hours.

So I had the format for the time-difference I wanted, but the place for the setTimeDiff function - which was inside the ItemDTO class - was not quiet right. 

To create the ItemDTO I already had a ItemDTOMapper class, so I also implemented the setTimeDiff function there:

```
public ItemDTO apply(Item item) {
        ItemDTO itemDTO = new ItemDTO(
                item.getId(),
                item.getName(),
                item.getExpirationDate(),
                item.getWastedDate(),
                item.getStorageLocation());

        PrettyTime prettyTime = new PrettyTime(Locale.forLanguageTag("en"));
        prettyTime.removeUnit(Hour.class);
        prettyTime.removeUnit(Minute.class);
        prettyTime.removeUnit(Second.class);
        prettyTime.removeUnit(Millisecond.class);
        var expirationDate = item.getExpirationDate();
        if (expirationDate == null) {
            itemDTO.setTimeDiff("");
        } else {
            var prettyTimeDiff = prettyTime.format(prettyTime.calculatePreciseDuration(expirationDate));
            itemDTO.setTimeDiff(prettyTimeDiff);
        }
        return itemDTO;
    }

```
    
Finally, I integrated the time difference calculation into the ItemService class:
```
public List<ItemDTO> findOrderedList(Long id) {
return repository.findByStorageLocationIdAndWastedDateIsNullOrderByExpirationDateAsc(id)
.stream()
.map(itemDTOMapper)
.collect(Collectors.toList());
}
```
This streamlined process ensures the creation of ItemDTO instances with simultaneous setting of the time difference.