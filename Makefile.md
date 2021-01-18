# Makefile

```
CROSS_COMPILE	?= arm-linux-gnueabihf-
TARGET			?= bsp

CC				:= $(CROSS_COMPILE)gcc
LD				:= $(CROSS_COMPILE)ld
OBJCOPY			:= $(CROSS_COMPILE)objcopy
OBJDUMP			:= $(CROSS_COMPILE)objdump

# INCDIRS 包含整个工程的.h 头文件目录，文件中的所有头文件目录都要添加到变量INCDIRS中
INCDIRS			:= imx6ul \
					bsp\clk \
					bsp\led \
					bsp\delay 
# SRCDIRS 包含的是整个工程的所有.c 和.S 文件目录
SRCDIRS			:= project \
					bsp\clk \
					bsp\led \
					bsp\delay
#通过函数 patsubst 给变量 INCDIRS 添加一个“-I”，即INCLUDE := -I imx6ul -I bsp/clk -I bsp/led -I bsp/delay
#加“-I”的目的是因为 Makefile 语法要求指明头文件目录的时候需要加上“-I”。
INCLUDE			:= $(patsubst %, -I %, $(INCDIRS))
#SFILES 保存工程中所有的.s 汇编文件
#变量 SRCDIRS 已经存放了工程中所有的.c 和.S 文件，
#所以我们只需要借助了函数 foreach 和函数 wildcard从里面挑出所有的.S 汇编文件即可
SFILES			:= $(foreach dir, $(SRCDIRS), $(wildcard $(dir)/*.S))
CFILES			:= $(foreach dir, $(SRCDIRS), $(wildcard $(dir)/*.c))
# SFILENDIR 和 CFILENDIR 包含所有的.S 汇编文件和.c 文件
#相比变量 SFILES 和 CFILES， SFILENDIR 和 CFILNDIR 只是文件名，不包含文件的绝对路径。
#使用函数 notdir 将 SFILES 和 CFILES 中的路径去掉即可
#SFILES := project/start.S
#CFILES = project/main.c bsp/clk/bsp_clk.c bsp/led/bsp_led.c bsp/delay/bsp_delay.c
#SFILENDIR = start.S
#CFILENDIR = main.c bsp_clk.c bsp_led.c bsp_delay.c
SFILENDIR		:= $(notdir $(SFILES))
CFILENDIR		:= $(notdir $(CFILES))
#变量 SOBJS 和 COBJS 是.S 和.c 文件编译以后对应的.o 文件目录
#SOBJS = obj/start.o
#COBJS = obj/main.o obj/bsp_clk.o obj/bsp_led.o obj/bsp_delay.o
SOBJS			:= $(patsubst %,obj/%, $(SFILEDIR:.S=.o))
COBJS			:= $(patsubst %,obj/%, $(CFILEDIR:.c=.o))
#变量 OBJS 是变量 SOBJS 和 COBJS 的集合
#OBJS = obj/start.o obj/main.o obj/bsp_clk.o obj/bsp_led.o obj/bsp_delay.o
OBJS			:= $(SOBJS) $(COBJS)
#VPATH 是指定搜索目录的，这里指定的搜素目录就是变量 SRCDIRS 所保存的目录，
#这样当编译的时候所需的.S 和.c 文件就会在 SRCDIRS 中指定的目录中查找
VPATH			:= $(SRCDIRS)

.PHONY:	clean

$(TARGET).bin : $(OBJS)
	$(LD) -Timx6ul.lds -o $(TARGET).elf $^
	$(OBJCOPY) -O binary -S $(TARGET).elf $@
	$(OBJDUMP) -D -m arm $(TARGET).elf > $(TARGET).dis

$(SOBJS) : obj/%.o : %.S
	$(CC) -Wall -nostdlib -c -O2 $(INCLUDE) -o $@ $<
	
$(COBJS) : obj/% : %.c
	$(CC) -Wall -nostdlib -c -O2 $(INCLUDE) -o $@ $<

clean:
	rm -rf $(TARGET).elf $(TARGET).dis $(TARGET).bin $(COBJS) $(SOBJS)

```

