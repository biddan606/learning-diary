# Visitor 패턴을 이용한 mapping

프로젝트에서 상속된 객체들을 `DTO` 로 매핑해야 하는 경우가 있었습니다.   
`if-else` 를 이용하면 되지만 코드가 깔끔하지 않았습니다. 당장에 타입이 2개밖에 없어 큰 문제는 없었지만, 깔끔한 방식으로 작성하는 패턴이 있을까에 대해 궁금하여 찾아보게 되었습니다.   
`baeldung` 에서 `Visitor` 패턴을 이용하는 방법에 대해 보게 되었고, 이를 정리한 글입니다.   

## 나의 상황

![alt text](<image/Visitor 패턴을 이용한 mapping/나의 상황.png>)

`Comment` 클래스를 상속받은 `MemberComment`, `GuestComment` 가 있습니다.   
이 객체들은 `CommentInfoMapper`를 통해 `CommentInfo` 로 매핑되어야 합니다.   
하지만, 상속받은 클래스들은 필드가 추가로 있어 `instanceof` 를 통해 다운캐스팅 후 매핑이 됩니다.   
이러한 코드는 추가 상속 관계가 생겼지만, `CommentInfoMapper` 에는 매핑하는 코드를 추가해주지 않는 경우 런타임 에러가 발생합니다.   
또한, 다운캐스팅으로 인한 약간의 성능 저하도 있습니다.   

## Visitor 패턴을 이용하여 해결하기

`baeldung` 에 예시와 `Visitor` 패턴을 사용하여 이러한 문제를 해결해보겠습니다.   

### baeldung 예제를 상속 관계

![alt text](<image/Visitor 패턴을 이용한 mapping/baeldung 구조.png>)

### Visitor 패턴 사용 전 if-else로 매핑하기

![alt text](<image/Visitor 패턴을 이용한 mapping/밸덩 예제 if-else 해결법.png>)

`baeldung` 예제도 저의 경우와 같이 `if-else` 로 해결할 수 있습니다. `mapstruct` 를 이용하여 생성되었고 코드는 다음과 같습니다.

```java
public class VehicleMapperBySubclassMappingImpl implements VehicleMapperBySubclassMapping {

    @Override
    public VehicleDTO mapToVehicleDTO(Vehicle vehicle) {
        if ( vehicle == null ) {
            return null;
        }

        if (vehicle instanceof Car) {
            return carToCarDTO( (Car) vehicle );
        }
        else if (vehicle instanceof Bus) {
            return busToBusDTO( (Bus) vehicle );
        }
        else {
            VehicleDTO vehicleDTO = new VehicleDTO();

            vehicleDTO.setColor( vehicle.getColor() );
            vehicleDTO.setSpeed( vehicle.getSpeed() );

            return vehicleDTO;
        }
    }

    protected CarDTO carToCarDTO(Car car) {
        if ( car == null ) {
            return null;
        }

        CarDTO carDTO = new CarDTO();

        carDTO.setColor( car.getColor() );
        carDTO.setSpeed( car.getSpeed() );
        carDTO.setTires( car.getTires() );

        return carDTO;
    }

    protected BusDTO busToBusDTO(Bus bus) {
        if ( bus == null ) {
            return null;
        }

        BusDTO busDTO = new BusDTO();

        busDTO.setColor( bus.getColor() );
        busDTO.setSpeed( bus.getSpeed() );
        busDTO.setCapacity( bus.getCapacity() );

        return busDTO;
    }
}
```

### Visitor 패턴 사용하기

![alt text](<image/Visitor 패턴을 이용한 mapping/visitor 패턴 구조.png>)

이전 구조와 다르게 다소 복잡합니다. 전체 코드를 보기 전에, 먼저 `Vehicle`, `Visitor`, `VehicleDTO` 가 어떻게 연결되어 있는지 알아보겠습니다.

```java
@Data
public abstract class Vehicle {

    private String color;
    private String speed;

    public abstract VehicleDTO accept(Visitor visitor);
}

public interface Visitor {

    VehicleDTO visit(Car car);

    VehicleDTO visit(Bus bus);
}

@Data
public class VehicleDTO {

    private String color;
    private String speed;

}
```

`Vehicle` 이 `Visitor` 를 호출하여 변환됩니다. 이는 `Car` -> `CarDTO`, `Bus`-> `BusDTO`로 매핑되는 책임을 `Mapper` 에서 `Car`, `Bus` 로 위임했다고 보면 이해하기 쉽습니다.   
메서드 호출이 `if-else` 코드에서는 `Mapper` 가 끝이지만, `Visitor` 코드에서는 `Mapper` -> `Vehicle` -> `Mapper` 순으로 수행됩니다.(결국 `Vehicle` 이 어떤 타입인지를 알아야 하는데, `instanceof` 를 통해 아느냐, 직접 객체 호출을 통해 아느냐의 차이)

```java
@Mapper()
public abstract class VehicleMapperByVisitorPattern implements Visitor {

    public VehicleDTO mapToVehicleDTO(Vehicle vehicle) {
        return vehicle.accept(this);
    }

    @Override
    public VehicleDTO visit(Car car) {
        return map(car);
    }

    @Override
    public VehicleDTO visit(Bus bus) {
        return map(bus);
    }

    abstract CarDTO map(Car car);

    abstract BusDTO map(Bus bus);
}

@EqualsAndHashCode(callSuper = true)
@Data
public class Bus extends Vehicle {

    private Integer capacity;

    @Override
    public VehicleDTO accept(Visitor visitor) {
        return visitor.visit(this);
    }
}

@EqualsAndHashCode(callSuper = true)
@Data
public class Car extends Vehicle {

    private Integer tires;

    @Override
    public VehicleDTO accept(Visitor visitor) {
        return visitor.visit(this);
    }
}

public class VehicleMapperByVisitorPatternImpl extends VehicleMapperByVisitorPattern {

    @Override
    CarDTO map(Car car) {
        if ( car == null ) {
            return null;
        }

        CarDTO carDTO = new CarDTO();

        carDTO.setColor( car.getColor() );
        carDTO.setSpeed( car.getSpeed() );
        carDTO.setTires( car.getTires() );

        return carDTO;
    }

    @Override
    BusDTO map(Bus bus) {
        if ( bus == null ) {
            return null;
        }

        BusDTO busDTO = new BusDTO();

        busDTO.setColor( bus.getColor() );
        busDTO.setSpeed( bus.getSpeed() );
        busDTO.setCapacity( bus.getCapacity() );

        return busDTO;
    }
}
```

`VehicleMapperByVisitorPattern.mapToVehicleDTO()` -> `Vehicle.accept()` -> `Visitor.visit()` -> `VehicleMapperByVisitorPattern.map()`

## 결론

- `Visitor` 패턴을 이용하면 매핑시 발생하는 런타임 에러를 컴파일 에러로 바꿀 수 있다.
- 다만 객체가 어떤 `DTO` 로 매핑되는지 알고 있어야 하고, 구조가 다소 복잡하다.
- 상속 관계가 적을 경우에는 `DTO` 매핑을 분리하거나 `if-else` 를 이용하고, 틀이 갖춰지고 상속 관계가 많아질 경우 사용하는 것이 좋을 것 같다.

## 참조

- [상속과 함께 MapStruct 사용하기](https://www.baeldung.com/java-mapstruct-inheritance)

- [Visitor 패턴 에제 코드](https://github.com/eugenp/tutorials/tree/master/mapstruct)
