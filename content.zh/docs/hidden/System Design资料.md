# 患者预约系统
## 功能需求
医生：设置可用时间段。

患者：查看可用时间段并预约。

系统：返回包含预约时段的字典。
## 系统架构
系统采用简化的架构，主要包括以下组件：

数据存储：使用 Map 存储医生的可用时间段和预约信息。

预约管理：处理预约的创建和查询。
## Java 实现示例
```java
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.*;

class AppointmentScheduler {
    private Map<String, List<TimeSlot>> doctorAvailability = new HashMap<>();
    private Map<String, List<TimeSlot>> appointments = new HashMap<>();

    // 添加医生的可用时间段
    public void addDoctorAvailability(String doctorId, List<TimeSlot> slots) {
        doctorAvailability.put(doctorId, slots);
    }

    // 获取医生的可用预约时间段
    public List<TimeSlot> getAvailableSlots(String doctorId) {
        List<TimeSlot> available = new ArrayList<>();
        List<TimeSlot> allSlots = doctorAvailability.getOrDefault(doctorId, new ArrayList<>());
        List<TimeSlot> bookedSlots = appointments.getOrDefault(doctorId, new ArrayList<>());

        for (TimeSlot slot : allSlots) {
            if (!bookedSlots.contains(slot)) {
                available.add(slot);
            }
        }
        return available;
    }

    // 为患者预约指定的时间段
    public boolean bookAppointment(String doctorId, String patientId, TimeSlot desiredSlot) {
        List<TimeSlot> availableSlots = getAvailableSlots(doctorId);
        if (availableSlots.contains(desiredSlot)) {
            appointments.computeIfAbsent(doctorId, k -> new ArrayList<>()).add(desiredSlot);
            System.out.println("预约成功：" + desiredSlot);
            return true;
        } else {
            System.out.println("预约失败，时间段不可用。");
            return false;
        }
    }

    // 获取医生的所有预约
    public List<TimeSlot> getAppointments(String doctorId) {
        return appointments.getOrDefault(doctorId, new ArrayList<>());
    }
}

class TimeSlot {
    private LocalDateTime start;
    private LocalDateTime end;

    public TimeSlot(LocalDateTime start, LocalDateTime end) {
        this.start = start;
        this.end = end;
    }

    // 重写 equals 和 hashCode 方法，以便在列表中正确比较 TimeSlot 对象
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (!(obj instanceof TimeSlot)) return false;
        TimeSlot other = (TimeSlot) obj;
        return start.equals(other.start) && end.equals(other.end);
    }

    @Override
    public int hashCode() {
        return Objects.hash(start, end);
    }

    @Override
    public String toString() {
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm");
        return start.format(formatter) + " - " + end.format(formatter);
    }
}

// 示例使用
public class Main {
    public static void main(String[] args) {
        AppointmentScheduler scheduler = new AppointmentScheduler();
        String doctorId = "Dr_Smith";
        String patientId = "Patient_1";

        // 创建医生的可用时间段
        List<TimeSlot> slots = new ArrayList<>();
        LocalDateTime startTime = LocalDateTime.of(2025, 5, 14, 9, 0);
        for (int i = 0; i < 8; i++) {
            TimeSlot slot = new TimeSlot(startTime.plusMinutes(i * 30), startTime.plusMinutes((i + 1) * 30));
            slots.add(slot);
        }

        // 添加医生的可用时间段
        scheduler.addDoctorAvailability(doctorId, slots);

        // 获取并显示可用时间段
        List<TimeSlot> availableSlots = scheduler.getAvailableSlots(doctorId);
        System.out.println("可用时间段：");
        for (TimeSlot slot : availableSlots) {
            System.out.println(slot);
        }

        // 预约一个时间段
        TimeSlot desiredSlot = availableSlots.get(0);
        scheduler.bookAppointment(doctorId, patientId, desiredSlot);

        // 显示所有预约
        List<TimeSlot> appointments = scheduler.getAppointments(doctorId);
        System.out.println("所有预约：");
        for (TimeSlot appointment : appointments) {
            System.out.println(appointment);
        }
    }
}
```
## 说明
医生可用时间段：通过 addDoctorAvailability 方法添加，系统会根据指定的时间范围生成所有可用的时间段。

获取可用时间段：getAvailableSlots 方法返回医生当前未被预约的时间段。

预约时间段：bookAppointment 方法允许患者预约一个指定的时间段，如果该时间段可用，则预约成功。

查看所有预约：getAppointments 方法返回医生的所有预约信息。