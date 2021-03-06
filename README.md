## S — The Single Responsibility Principle (SRP):

### Bad

```kotlin
fun main() {
    val users = listOf(User(name = "Android User", mobileNumbers = listOf("987654321", "9999999999")))
    val adapter = Adapter(users)
}

class Adapter(private val users: List<User>) {
    
    /** Calling onBindViewHolder here just for a demo purpose. In a RecyclerView adapter,
     *  onBindViewHolder will be triggered as part of adapter life cycle.
     **/  
    init {
        onBindViewHolder(0)
    }
    
    fun onBindViewHolder(position: Int) {
        val user = users[position]
        
        println("Name has been set as ${user.name} to TextView")
        
        /** 
         * I didn't receive the proper data to display it in the UI.. 
         * So let me convert it.
        **/
        val mobileNumber = user.mobileNumbers.joinToString()
        
        println("Mobile numbers has been set as $mobileNumber to TextView")
    }
}


data class User(val name: String, val mobileNumbers: List<String>)
```

### Good

```kotlin

fun main() {
    val users = listOf(User(name = "Android User", mobile = "987654321, 9999999999"))
    val adapter = Adapter(users)
}

class Adapter(private val users: List<User>) {
    
    /** Calling onBindViewHolder here just for a demo purpose,
     *  onBindViewHolder will be triggered as part of adapter life cycle.
     **/  
    init {
        onBindViewHolder(0)
    }
    
    fun onBindViewHolder(position: Int) {
        val user = users[position]
        
        println("Name has been set as ${user.name} to TextView")
        println("Mobile number has been set as ${user.mobile} to TextView")
    }
}


data class User(val name: String, val mobile: String)
```
## O — The Open-Closed Principle (OCP)

### Bad

```kotlin
fun main() {
    val mileageCalculator = MileageCalculator()
    mileageCalculator.showMileage(Car())
}

class MileageCalculator {

    fun showMileage(anyView: Any) {
        when {
            anyView is Bike -> {
                print(anyView.getBikeMileage())
            }
            anyView is Car -> {
                print(anyView.getCarMileage())
            }
        }
    }
}


class Bike {
    fun getBikeMileage(): String = "50"
}

class Car {
    fun getCarMileage(): String = "12"
}
```

### Good

```kotlin
fun main() {
    val mileageCalculator = MileageCalculator()
    mileageCalculator.showMileage(Bike())
}

class MileageCalculator {

    fun showMileage(vehicle: Vehicle) {
        println(vehicle.getMileage())
    }
}

interface Vehicle {
    fun getMileage(): String
} 

class Bike: Vehicle {
    override fun getMileage(): String = "50"
}

class Car: Vehicle {
    override fun getMileage(): String = "12"
}
```

## L - The Liskov Substitution Principle (LSP)

### Bad
```kotlin
fun main() {
    val adapter = Adapter()
    adapter.select(RadioButton())
}

class Adapter {

    fun select(clickListener: ClickListener) {
        when {
           clickListener is ListItem -> {
               clickListener.changeTheBackground()
           }
           
           clickListener is RadioButton -> {
               clickListener.check()
           }
        }
        clickListener.onClick(1)
    }
}

interface ClickListener {
    fun onClick(position: Int)
} 

class ListItem: ClickListener {
    override fun onClick(position: Int){
       println("Clicked ListItem $position")
    }
    
    fun changeTheBackground() {
       println("Change the background color of the item view")
    }
    
}

class RadioButton: ClickListener {
    override fun onClick(position: Int){
       println("Clicked RadioButton $position")
    }
    
    fun check() {
       println("Enable the radio button")
    }
}
```

### Good

```kotlin
fun main() {
    val adapter = Adapter()
    adapter.select(RadioButton())
}

class Adapter {

    fun select(clickListener: ClickListener) {
        clickListener.onClick(1)
    }
}

interface ClickListener {
    fun onClick(position: Int)
} 

class ListItem: ClickListener {
    override fun onClick(position: Int){
       changeTheBackground()
       println("Clicked ListItem $position")
    }
    
    fun changeTheBackground() {
       println("Change the background color of the item view")
    }
    
}

class RadioButton: ClickListener {
    override fun onClick(position: Int){
       check()
       println("Clicked RadioButton $position")
    }
    
    fun check() {
       println("Enable the radio button")
    }
}
```
## I — The Interface Segregation Principle (ISP):

### Bad
```kotlin
fun main() {
    val adapter = Adapter()
    adapter(object: Adapter.OnClickListener {
        override fun onItemClick(position: Int) {
            // Yes, I have received a callback, go to the next activity.
            println("Clicked position is $position")
        }
        override fun onRadioButtonClick(position: Int) {
            // This is no longer needed for this activity, but still I have been implemented for no use...
        }

    })
    adapter.execute()
}

class Adapter {

    private var onClickListener: OnClickListener? =null
   
    operator fun invoke (onClickListener: OnClickListener) {
        this.onClickListener = onClickListener
    }

    fun execute() {
        onClickListener?.onItemClick(4)
    }
    
    interface OnClickListener {
        fun onItemClick(position: Int)
        fun onRadioButtonClick(position: Int)
    }
}

```
### Good

```kotlin
fun main() {
    val adapter = Adapter()
    adapter(object: Adapter.OnItemClickListener {
        override fun onItemClick(position: Int) {
            // Yes, I have received a callback, go to the next activity.
            println("Clicked position is $position")
        }
    })
    adapter.execute()
}

class Adapter {

    private var onItemClickListener: OnItemClickListener? =null
   
    operator fun invoke (onItemClickListener: OnItemClickListener) {
        this.onItemClickListener = onItemClickListener
    }

    fun execute() {
        onItemClickListener?.onItemClick(4)
    }
    
    interface OnItemClickListener {
        fun onItemClick(position: Int)
    }
    
    interface OnRadioClickListener {
        fun onRadioButtonClick(position: Int)
    }

}
```

## D - The Dependency Inversion Principle (DIP)

### Bad
```kotlin
fun main() {
    val user = User()
    user.getMyData(3)
}

class User {

    fun getMyData(requestCode: Int) {
        when (requestCode) {
            1 -> {
                val firebase = Firebase()
                firebase.fetchData() 
            }
            2 -> {
                val restClient = RestClient()
                restClient.fetchData() 
            } 
            else -> print("I dont care about the user. Don't do anything. Keep quiet!")
        } 

    }
}

class Firebase {
    fun fetchData(){
        print("Syncing the data from the firebase storage")
    } 
}

class RestClient {
    fun fetchData(){ 
        print("Hitting the api and getting back the response")
    } 
}
```
### Good
```kotlin
fun main() {
    /* As a end user, I don't care about how will I retrieve the data. 
     * Do whatever you want and simply get my data. 
     */
    val user = User(dataFetcher = Firebase())
    user.getMyData()
}

class User(private val dataFetcher: DataFetcher) {

    fun getMyData() {
        dataFetcher.fetchData()
    }
}

interface DataFetcher {
    fun fetchData()
}

class Firebase: DataFetcher {
    override fun fetchData(){
       print("Syncing the data from the firebase storage")
    } 
}

class RestClient: DataFetcher {
    override fun fetchData(){ 
       print("Hitting the api and getting back the response")
    } 
}
```
