// projectb

void printString(char*);
void readString(char*);
void readSector(char*,int);
void handleInterrupt21(int, int, int, int);

void main() {
	char line [80];
	char buffer[512];
	
	//printString("Enter a line:");
	//readString(line);
	//printString(line);
	
	//readSector(buffer, 30);
	//printString(buffer);
	
	makeInterrupt21();   
	//is used to read a line of text from the user and store it in the line array
 	interrupt(0x21,1,line,0,0);
  	//is used to print the contents of the line array
	interrupt(0x21,0,line,0,0);
 	//is used to read a sector from a storage device and store it in the buffer array
	interrupt(0x21,2,buffer,30,0);
 	//is used to print the contents of the buffer array
	interrupt(0x21,0,buffer,0,0);
 	//while keep the program running
	while(1);
}
	//printString function is used to print a string to the screen character by character, It takes a character pointer chars and uses interrupt 0x10 (video services) to display each character
void printString(char* chars) {
	int i=0;
	while(chars[i] != 0x0) {
		interrupt(0x10, 0xe*256+chars[i++], 0, 0, 0);
	}
}

	//readString function reads a line of text from the user
void readString(char* chars) {
	char inp = interrupt(0x16, 0, 0, 0, 0);
	int i=0;
	while(inp != 0xd) {
		chars[i++] = inp;
		inp = interrupt(0x16, 0, 0, 0, 0);
	}
	chars[i++] = 0xa;
	chars[i++] = 0x0;
}	
//readSector function reads a sector from a storage device
void readSector(char* chars, int sector) {
	int AH = 2;
	int AL = 1;
	int AX = AH*256+AL;
	int BX = chars;
	int CH = 0;
	int CL = sector+1;
	int CX = CH*256+CL;
	int DH = 0;
	int DL = 0x80;
	int DX = DH*256+DL;

	interrupt(0x13, AX, BX, CX, DX);
}
	//The handleInterrupt21 function is called by the interrupt handler and serves as a dispatcher.Depending on the value of ax, it calls one of the printString, readString, or readSector
void handleInterrupt21(int ax, int bx, int cx, int dx) {
	if( ax == 0) {
		printString(bx);
	} else if(ax == 1) {
		readString(bx);
	} else if(ax == 2) {
		readSector(bx, cx);
	} else {
		printString("AX must be one of 1,2,3");
	}
}	
