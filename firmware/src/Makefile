BUILD_DIR=../build
DIST_DIR=../dist

BIN=sdisk2_oled
#BIN=sdisk2_original_mod_nokia
#BIN=sdisk2_original

BUILD_TYPE=_SDISK_OLED_
#BUILD_TYPE=_SDISK_ORIGINAL_MOD_NOKIA_
#BUILD_TYPE=_SDISK_ORIGINAL_ 

CC=avr-gcc

DEPFLAGS = -MT $@ -MMD -MP -MF $(BUILD_DIR)/$*.Td
DEPDIR=$(BUILD_DIR)

CFLAGS=-Wall -Os -Wno-deprecated-declarations -Wno-strict-aliasing -D__PROG_TYPES_COMPAT__ 
CFLAGS+=-D$(BUILD_TYPE) -fpack-struct -fshort-enums -std=gnu99 -funsigned-char -funsigned-bitfields 
CFLAGS+=-mmcu=atmega328p -DF_CPU=25000000UL $(DEPFLAGS) -c

ASFLAGS=-x assembler-with-cpp -mmcu=atmega328p $(DEPFLAGS) -D$(BUILD_TYPE) -c
LDFLAGS=-Wl,-Map,$(BUILD_DIR)/$(BIN).map -mmcu=atmega328p -D$(BUILD_TYPE) 
OBJCOPY=avr-objcopy

FILES=oled.o SPI_routines.o SD_routines.o FAT32.o sub.o main.o
#FILES=lcd_nokia.o SPI_routines.o SD_routines.o FAT32.o sub.o main.o
#FILES=lcd.o SPI_routines.o SD_routines.o FAT32.o sub.o main.o
OBJS=$(addprefix $(BUILD_DIR)/, $(FILES))

SRCS=$(OBJS:%.o=%.c)

vpath %.o $(BUILD_DIR)
vpath %.elf $(BUILD_DIR)
vpath %.hex $(DIST_DIR)

POSTCOMPILE = @mv -f $(DEPDIR)/$*.Td $(DEPDIR)/$*.d && touch $@

.PHONY : all
all: $(DIST_DIR)/$(BIN).hex

$(BUILD_DIR)/%.elf : $(OBJS)
	$(CC) $(LDFLAGS) -o $@ $^

$(BUILD_DIR)/%.o : %.c
$(BUILD_DIR)/%.o : %.c $(DEPDIR)/%.d
	$(CC) $(CFLAGS) -o $@ $<
	$(POSTCOMPILE)

$(BUILD_DIR)/%.o : %.S
$(BUILD_DIR)/%.o : %.S $(DEPDIR)/%.d
	$(CC) $(ASFLAGS) -c -o $@ $<
	$(POSTCOMPILE)

$(DIST_DIR)/$(BIN).hex : $(BUILD_DIR)/$(BIN).elf
	$(OBJCOPY) -O ihex -R .eeprom $< $@
	avr-size --format=avr --mcu=atmega328p $(BUILD_DIR)/$(BIN).elf

ifeq ($(OS),Windows_NT)
    RM = cmd //C del //Q //F
    RRM = cmd //C rmdir //Q //S
else
    RM = rm -f
    RRM = rm -f -r
endif

SRC=main.c

$(DEPDIR)/%.d : ;

.PRECIOUS: $(BUILD_DIR)/%.o $(DEPDIR)/%.d
include $(wildcard $(patsubst %,$(DEPDIR)/%.d,$(basename $(SRCS))))

.PHONY : clean
clean:
	$(RRM) $(BUILD_DIR)/*.o
	$(RRM) $(BUILD_DIR)/*.d
	$(RRM) $(BUILD_DIR)/*.elf
	
.PHONY : distclean
distclean: clean
	$(RRM) $(DIST_DIR)/*.hex

# autodependency based upon Paul D. Smith's work
# http://make.mad-scientist.net/papers/advanced-auto-dependency-generation
