MEMSIZE32=1073741824

OLEVEL=-O2
CC32FLAGS=-DSKIP32 --target=wasm32 -emit-llvm
CC64FLAGS=$(OLEVEL) -DSKIP64

CFILES=\
	copy.c \
	free.c \
	hash.c \
	hashtable.c \
	intern.c \
	memory.c \
	obstack.c \
	runtime.c \
	stdlib.c \
	stack.c \
	string.c \
	native_eq.c

NATIVE_FILES=\
	palloc.c\
	consts.c

CFILES32=$(CFILES) runtime32_specific.c
CFILES64=$(CFILES) runtime64_specific.cpp $(NATIVE_FILES)
BCFILES32=magic.bc $(CFILES32:.c=.bc)
OFILES=$(CFILES:.c=.o)
ONATIVE_FILES=magic.o $(NATIVE_FILES:.c=.o)

.PHONY: default
default: libskip_runtime64.a libskip_runtime32.bc

magic.c:
	date | cksum | awk '{print "unsigned long version = " $$1 ";"}' > magic.c
	echo "int SKIP_get_version() { return (int)version; }" >> magic.c

magic.bc: magic.c
	clang $(OLEVEL) $(CC32FLAGS) -o $@ -c $<

magic.o: magic.c
	clang $(CC64FLAGS) -o $@ -c $<

libskip_runtime32.bc: $(BCFILES32)
	llvm-link $(BCFILES32) -o $@

%.bc: %.c
	clang $(OLEVEL) $(CC32FLAGS) -o $@ -c $<

libskip_runtime64.a: $(OFILES) runtime64_specific.o $(ONATIVE_FILES)
	$(AR) rcs $@ $^

runtime64_specific.o: runtime64_specific.cpp
	clang++ $(OLEVEL) -o $@ -c $<

%.o: %.c
	clang $(CC64FLAGS) -o $@ -c $<

.PHONY: clean
clean:
	rm -f *.o *.bc magic.c libskip_runtime64.a