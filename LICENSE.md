///////////////////////////////////////INCLUINDO BIBLIOTECAS & DEFININDO PORTAS//////////////////////////////////////////////////
#include <Adafruit_GFX.h>    // Core graphics library
#include <Adafruit_ST7789.h> // Hardware-specific library for ST7789
#include <string.h>
#include <SPI.h>             // Arduino SPI library
#include <EEPROM.h>

#define TFT_CS     10
#define TFT_RST    8  // define reset pin, or set to -1 and connect to Arduino RESET pin
#define TFT_DC     9

static unsigned long frameCount;
Adafruit_ST7789 tft = Adafruit_ST7789(TFT_CS, TFT_DC, TFT_RST);




///////////////////////////////////////INICIALIZACAO DAS VARIAVEIS//////////////////////////////////////////////////
const char versao[] = "1.01.1";                    // Apenas a versao
const short w = 3, s = 4, a = 5, d = 6;           // VARIAVEL PARA A LEITUDO DO PUSHBUTTON
short cursor_y = 0;                              // cursor_y - movimentacao e escolha do menu.
short loading_line = 0;                         // Variavel loading_line para ser incrementada gerando o aumento da barra de loading no inicio
short ASCII_NUM = 65;                          // Numero ASCII representando a letra "A", para a gravura do nome do player
short add_x = 240;                            // Variavel para movimentacao nos creditos
short uni_char = 0;                          // Auxilia para salvar o nome
short vet_nome;                             // Identifica a posicao do save abc = 0;
int cursor_x = 0;                          // Variavel cursor_x para auxiliar no posicionamento horizontal
int cursor_yJogo = 0;                     // Movimentacao do personagem.
int flecha_P,hitbox_inimigo;             // Variavel x do ataque do personagem principal
int pontos = 0;                         // Variavel de controle de pontos do jogador
int vidas = 4;                         // Variavel de controle de vidas restantes do jogador
unsigned long tempo1 = millis();      // Variavel para controlar o tempo
int aleatorio;                       // Escolhe aleatoriamente onde os inimigos irao spawnar
unsigned long tempo3 = millis();    // Variavel para controlar o tempo
short endereco=0; 
char nome_aux[3];
int tiro1,tiro2,tiro3,tiro4;
int inimigoY,flecha_I,flecha_II,flecha_III,flecha_IV;
bool inimigo;
char abc;
int velocidade=1;
unsigned long Vtempo;

//////////////////DECLARACAO DAS FUNCOES//////////////////////////
void draw_inicio();                     // Animacao inicial
void tela_inicio();                    // Tela de inicio
void cursor();                        // Altera a posicao do cursor apagando a ultima
void add_nome();                     // Adicionar nome ao jogador
void jogo();                        // Jogo em si
void jo_salvamento();              // Mostra o fin de jogo e armazena o nome       
void rank();                      // Mostra o ranking dos jogadores
void creditos();                 // Mostra os creditos
void sair();                    // Apaga a tela e armazena na EEPROM
void inimigos();               // Printa os inimigos na tela
void flecha_inimigo();        // Printa o ataque dos inimigos
void flecha_Personagem();    // Printa o ataque do personagem 
void hitbox_inimigos();     // Verifica se os inimigos foram atingidos
void loading_screem();      
void cursor();
void hitbox_personagem();
void dificuldade();

///////////////////////////////////////BITMAP personagem/////////////////////

const unsigned char berglot [] PROGMEM = {
  0x00, 0x3f, 0x00, 0x00, 0x00, 0x79, 0x80, 0x00, 0x00, 0xf0, 0x80, 0x00, 0x01, 0xf0, 0x80, 0x00, 
  0x03, 0xf0, 0x8c, 0x00, 0x03, 0x70, 0x8e, 0x00, 0x21, 0x39, 0x87, 0x00, 0x40, 0x19, 0x03, 0x80, 
  0xb8, 0x09, 0x01, 0x80, 0x38, 0x00, 0x01, 0x80, 0x01, 0xff, 0xfd, 0x80, 0x18, 0xff, 0xff, 0x80, 
  0x18, 0x00, 0x01, 0x80, 0x19, 0xbd, 0x81, 0x80, 0x1f, 0xbd, 0x81, 0x80, 0x18, 0x3d, 0x83, 0x80, 
  0x19, 0xfd, 0x87, 0x00, 0x19, 0xfd, 0x8e, 0x00, 0x03, 0xf9, 0x8c, 0x00, 0x01, 0xf1, 0x80, 0x00, 
  0x07, 0xf1, 0x80, 0x00, 0x07, 0xf1, 0x80, 0x00, 0x1f, 0xf1, 0x80, 0x00, 0x1f, 0xe0, 0x00, 0x00, 
  0x00, 0x00, 0x00, 0x00, 0x01, 0x81, 0x80, 0x00, 0x01, 0x81, 0x80, 0x00, 0x01, 0xc1, 0xc0, 0x00, 
  0x01, 0xc1, 0xc0, 0x00, 0x00, 0x00, 0x00, 0x00
};

const unsigned char wizardanim [] PROGMEM = {
  0x00, 0x00, 0x00, 0x00, 0x0f, 0x00, 0x00, 0x0c, 0x00, 0x00, 0x0e, 0x00, 0x70, 0x3f, 0x00, 0x50, 
  0x7f, 0x80, 0x10, 0x00, 0x00, 0x33, 0xff, 0xf8, 0x23, 0xff, 0xfc, 0x20, 0x00, 0x00, 0x00, 0x00, 
  0x00, 0x70, 0xe7, 0x80, 0x74, 0xe7, 0xe0, 0x76, 0x61, 0xf0, 0x06, 0x67, 0xf8, 0x22, 0xe7, 0xb8, 
  0x60, 0xc7, 0xb8, 0x40, 0xc4, 0x80, 0x40, 0xc7, 0xb8, 0x40, 0xc7, 0x80, 0x40, 0xc7, 0x80, 0x40, 
  0x00, 0x00, 0x40, 0x61, 0x80, 0x40, 0x61, 0x80, 0x00, 0xe1, 0xc0, 0x00, 0xc0, 0xc0, 0x00, 0x00, 
  0x00, 0x01, 0xc1, 0xc0, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00
};

// 'pilar_baixo', 255x15px
const unsigned char pilar_baixo [] PROGMEM = {
  0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 
  0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 
  0x20, 0x40, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 
  0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x04, 0x08, 
  0x20, 0x80, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 
  0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x08, 
  0x3e, 0x20, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 
  0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x08, 0xf8, 
  0x20, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 
  0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x08, 
  0x21, 0x10, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 
  0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x11, 0x08, 
  0x38, 0x8f, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 
  0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xe2, 0x38, 
  0x3c, 0x4f, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 
  0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xe4, 0x78, 
  0x22, 0x28, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 
  0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x28, 0x88, 
  0x20, 0x28, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 
  0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x28, 0x08, 
  0x39, 0xaf, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 
  0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xe9, 0x38, 
  0x21, 0x08, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 
  0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x21, 0x08, 
  0x2d, 0x28, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 
  0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x29, 0x68, 
  0x35, 0x5d, 0x55, 0x55, 0x55, 0x55, 0x55, 0x55, 0x55, 0x55, 0x55, 0x55, 0x55, 0x55, 0x55, 0x55, 
  0x55, 0x55, 0x55, 0x55, 0x55, 0x55, 0x55, 0x55, 0x55, 0x55, 0x55, 0x55, 0x55, 0x55, 0x7d, 0x58, 
  0x3f, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 
  0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xf8
};
// 'pilar_dir', 12x255px
const unsigned char pilar_dir [] PROGMEM = {
  0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0xff, 0xd0, 0x0a, 0x40, 0x49, 0x40, 0x48, 0xc0, 0x49, 0xc0, 
  0x04, 0x40, 0x44, 0x40, 0x23, 0xc0, 0x00, 0x40, 0x88, 0x50, 0x45, 0xc0, 0x02, 0xc0, 0x30, 0x40, 
  0x20, 0x40, 0x0d, 0xc0, 0x0a, 0xc0, 0x0b, 0x40, 0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 
  0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 
  0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 
  0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 
  0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 
  0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 
  0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 
  0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 
  0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 
  0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 
  0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 
  0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 
  0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 
  0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 
  0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 
  0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 
  0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 
  0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 
  0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 
  0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 
  0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 
  0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 
  0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 
  0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 
  0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 
  0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 
  0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 
  0x0a, 0x40, 0x19, 0x40, 0x0a, 0x40, 0x0b, 0x40, 0x0a, 0xc0, 0x0f, 0xc0, 0x20, 0x40, 0x00, 0x40, 
  0x47, 0xc0, 0x00, 0x40, 0x88, 0x50, 0x11, 0xc0, 0x23, 0xc0, 0x44, 0x40, 0x40, 0x40, 0x49, 0xc0, 
  0x08, 0x40, 0x4b, 0x40, 0xea, 0xd0, 0xff, 0xd0, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00
};
// 'pilar_esq', 9x255px
const unsigned char pilar_esq [] PROGMEM = {
  0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x7f, 0x80, 0x4a, 0x00, 0x42, 0x00, 0x62, 0x00, 0x73, 0x00, 
  0x44, 0x00, 0x44, 0x00, 0x78, 0x80, 0x51, 0x00, 0x42, 0x00, 0x74, 0x00, 0x68, 0x00, 0x41, 0x80, 
  0x40, 0x80, 0x7e, 0x00, 0x6a, 0x00, 0x5b, 0x00, 0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 
  0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 
  0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 
  0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 
  0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 
  0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 
  0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 
  0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 
  0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 
  0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 
  0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 
  0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 
  0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 
  0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 
  0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 
  0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 
  0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 
  0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 
  0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 
  0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 
  0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 
  0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 
  0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 
  0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 
  0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 
  0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 
  0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 
  0x4a, 0x00, 0x52, 0x00, 0x4a, 0x00, 0x5b, 0x00, 0x6a, 0x00, 0x7e, 0x00, 0x40, 0x80, 0x41, 0x00, 
  0x7c, 0x00, 0x40, 0x00, 0x42, 0x00, 0x71, 0x00, 0x78, 0x80, 0x44, 0x00, 0x40, 0x00, 0x73, 0x00, 
  0x42, 0x00, 0x5a, 0x00, 0x6a, 0x80, 0x7f, 0x80, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00
};
// 'nome_jogo', 144x43px
const unsigned char nome_jogo [] PROGMEM = {
  0x00, 0x06, 0x01, 0xc0, 0x0d, 0xff, 0x00, 0x70, 0x0f, 0xe0, 0x06, 0x03, 0xc0, 0x1c, 0x03, 0xf0, 
  0x01, 0x80, 0x00, 0x06, 0x00, 0xc0, 0x08, 0xe3, 0x00, 0x20, 0x30, 0x38, 0x06, 0x00, 0xc0, 0x08, 
  0x0c, 0x0c, 0x01, 0x80, 0x00, 0x0e, 0x00, 0xc0, 0x08, 0xe3, 0x80, 0x20, 0x60, 0x08, 0x06, 0x00, 
  0xe0, 0x08, 0x18, 0x04, 0x01, 0x80, 0x00, 0x36, 0x00, 0x60, 0x10, 0xe2, 0xc0, 0x20, 0xc0, 0x00, 
  0x0b, 0x00, 0xb0, 0x08, 0x30, 0x00, 0x02, 0xc0, 0x00, 0x36, 0x00, 0x60, 0x10, 0xe2, 0xc0, 0x20, 
  0xc0, 0x00, 0x0b, 0x00, 0xb0, 0x08, 0x30, 0x00, 0x02, 0xc0, 0x00, 0x46, 0x00, 0x60, 0x10, 0xe2, 
  0x70, 0x20, 0xc0, 0x00, 0x0b, 0x00, 0x9c, 0x08, 0x20, 0x00, 0x02, 0xc0, 0x00, 0x46, 0x00, 0x30, 
  0x20, 0xe2, 0x38, 0x21, 0x80, 0x00, 0x11, 0x80, 0x8e, 0x08, 0x60, 0x00, 0x04, 0x60, 0x00, 0x86, 
  0x00, 0x30, 0x20, 0xe2, 0x0c, 0x21, 0x80, 0x00, 0x11, 0x80, 0x83, 0x08, 0x60, 0x00, 0x04, 0x60, 
  0x01, 0xfe, 0x00, 0x18, 0x40, 0xe2, 0x0e, 0x21, 0x80, 0x00, 0x1f, 0x80, 0x83, 0x88, 0x60, 0x00, 
  0x07, 0xe0, 0x03, 0x06, 0x00, 0x18, 0x40, 0xe2, 0x06, 0x21, 0x80, 0x00, 0x20, 0xc0, 0x81, 0x88, 
  0x60, 0x00, 0x08, 0x30, 0x02, 0x06, 0x00, 0x18, 0x80, 0xe2, 0x03, 0x21, 0x80, 0x3c, 0x20, 0xc0, 
  0x80, 0xc8, 0x70, 0x00, 0x08, 0x30, 0x04, 0x06, 0x00, 0x0c, 0x80, 0xe2, 0x01, 0xa0, 0x80, 0x18, 
  0x20, 0xc0, 0x80, 0x68, 0x70, 0x00, 0x08, 0x30, 0x0c, 0x06, 0x00, 0x0c, 0x80, 0xe2, 0x00, 0xe0, 
  0xc0, 0x18, 0xc0, 0x70, 0x80, 0x38, 0x38, 0x00, 0x30, 0x1c, 0x08, 0x06, 0x00, 0x07, 0x00, 0xe2, 
  0x00, 0x60, 0x60, 0x18, 0xc0, 0x70, 0x80, 0x18, 0x1c, 0x01, 0x30, 0x1c, 0x10, 0x06, 0x00, 0x07, 
  0x00, 0xe2, 0x00, 0x20, 0x30, 0x19, 0x00, 0x78, 0x80, 0x08, 0x1e, 0x06, 0x40, 0x1e, 0xf0, 0x0f, 
  0x00, 0x04, 0x01, 0xf7, 0x00, 0x20, 0x0f, 0xff, 0xc0, 0x7d, 0xc0, 0x08, 0x07, 0xfc, 0xf0, 0x1f, 
  0xf0, 0x0f, 0x00, 0x04, 0x01, 0xf7, 0x00, 0x20, 0x0f, 0xff, 0xc0, 0x7d, 0xc0, 0x08, 0x07, 0xfc, 
  0xf0, 0x1f, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 
  0x01, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 
  0x00, 0x00, 0x01, 0xe0, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 
  0x00, 0x00, 0x00, 0x00, 0x00, 0xe0, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 
  0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x03, 0xc0, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 
  0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 
  0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 
  0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 
  0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 
  0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 
  0x00, 0x00, 0x00, 0x00, 0x3f, 0xfc, 0x0f, 0xfc, 0x01, 0xfe, 0x07, 0xfe, 0x7f, 0x80, 0x00, 0xfe, 
  0x3e, 0x00, 0x07, 0xe0, 0xff, 0xfc, 0x18, 0x3e, 0x03, 0x04, 0x00, 0xe3, 0x03, 0x02, 0x30, 0xe0, 
  0x03, 0x03, 0x9c, 0x00, 0x18, 0xf8, 0x43, 0x10, 0x18, 0x3e, 0x03, 0x04, 0x00, 0xe3, 0x03, 0x02, 
  0x30, 0xe0, 0x03, 0x03, 0x9c, 0x00, 0x18, 0xf8, 0x43, 0x10, 0x18, 0x07, 0x03, 0x04, 0x00, 0xe1, 
  0x83, 0x02, 0x30, 0x60, 0x06, 0x00, 0x9c, 0x00, 0x20, 0x1c, 0x03, 0x00, 0x18, 0x03, 0x83, 0x00, 
  0x00, 0xe1, 0x83, 0x00, 0x30, 0x60, 0x0c, 0x00, 0x1c, 0x00, 0x60, 0x1c, 0x03, 0x00, 0x18, 0x01, 
  0xc3, 0x00, 0x00, 0xe1, 0x83, 0x00, 0x30, 0x60, 0x0c, 0x00, 0x1c, 0x00, 0x40, 0x0e, 0x03, 0x00, 
  0x18, 0x00, 0xc3, 0x00, 0x00, 0xe1, 0x03, 0x00, 0x30, 0x40, 0x18, 0x00, 0x1c, 0x00, 0xc0, 0x06, 
  0x03, 0x00, 0x18, 0x00, 0xc3, 0x00, 0x00, 0xe2, 0x03, 0x00, 0x30, 0xc0, 0x18, 0x00, 0x1c, 0x00, 
  0xc0, 0x06, 0x03, 0x00, 0x18, 0x00, 0xc3, 0xf8, 0x00, 0xff, 0x83, 0xfc, 0x3f, 0x80, 0x18, 0x00, 
  0x1c, 0x00, 0xc0, 0x06, 0x03, 0x00, 0x18, 0x00, 0xc3, 0x00, 0x00, 0xe1, 0xc3, 0x00, 0x33, 0x80, 
  0x18, 0x00, 0x1c, 0x00, 0xc0, 0x06, 0x03, 0x00, 0x18, 0x00, 0xc3, 0x00, 0x00, 0xe0, 0xc3, 0x00, 
  0x30, 0xc0, 0x18, 0x03, 0xdc, 0x00, 0xc0, 0x06, 0x03, 0x00, 0x18, 0x01, 0x83, 0x00, 0x00, 0xe0, 
  0xc3, 0x00, 0x30, 0x60, 0x08, 0x01, 0x9c, 0x00, 0xe0, 0x04, 0x03, 0x00, 0x18, 0x01, 0x83, 0x00, 
  0x00, 0xe0, 0xc3, 0x00, 0x30, 0x60, 0x0c, 0x01, 0x9c, 0x00, 0x60, 0x0c, 0x03, 0x00, 0x18, 0x03, 
  0x03, 0x03, 0x00, 0xe0, 0xc3, 0x01, 0x30, 0x30, 0x06, 0x01, 0x9c, 0x08, 0x70, 0x18, 0x03, 0x00, 
  0x18, 0x03, 0x03, 0x03, 0x00, 0xe0, 0xc3, 0x01, 0x30, 0x30, 0x06, 0x01, 0x9c, 0x08, 0x70, 0x18, 
  0x03, 0x00, 0x1c, 0x06, 0x03, 0x03, 0x00, 0xe1, 0x83, 0x01, 0x30, 0x18, 0x03, 0x01, 0x9e, 0x08, 
  0x3c, 0x30, 0x03, 0x00, 0x3f, 0xfc, 0x0f, 0xff, 0x01, 0xff, 0x07, 0xff, 0x78, 0x0f, 0xc0, 0xff, 
  0xff, 0xf8, 0x0f, 0xc0, 0x07, 0x80
};
// 'pilar_cima', 255x17px
const unsigned char pilar_cima [] PROGMEM = {
  0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 
  0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 
  0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 
  0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 
  0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 
  0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 
  0x3f, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 
  0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xf8, 
  0x25, 0x08, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 
  0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x21, 0x48, 
  0x21, 0x28, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 
  0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x29, 0x28, 
  0x31, 0x0f, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 
  0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xe9, 0x18, 
  0x39, 0xaf, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 
  0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xe9, 0x38, 
  0x22, 0x28, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 
  0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x20, 0x88, 
  0x22, 0x28, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 
  0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x28, 0x88, 
  0x3c, 0x4f, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 
  0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xe4, 0x78, 
  0x28, 0x88, 0x88, 0x88, 0x88, 0x88, 0x88, 0x88, 0x88, 0x88, 0x88, 0x88, 0x88, 0x88, 0x88, 0x88, 
  0x88, 0x88, 0x88, 0x88, 0x88, 0x88, 0x88, 0x88, 0x88, 0x88, 0x88, 0x88, 0x88, 0x88, 0x80, 0x08, 
  0x21, 0x10, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 
  0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x11, 0x08, 
  0x3a, 0x20, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 
  0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x08, 0xb8, 
  0x34, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 
  0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x58, 
  0x20, 0xc0, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 
  0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x06, 0x08, 
  0x20, 0x40, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 
  0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x04, 0x08
};


////////STRUCT PARA GUARDAR O NOMES E PONTOS////////
typedef struct {
  char nome[10];
  int pontos;
} D;

D jogador[3] = {
  {"Novo Jogo", 0},
  {"Novo Jogo", 0},
  {"Novo Jogo", 0}
};

D aux[1]{

};

/////////////////////////////////////////////////VOID SETUP////////////////////////////////////////////////////////////
void setup(void) {
  Serial.begin(9600);
  tft.init(240, 240, SPI_MODE2);                //inicializando o display
  tft.fillScreen(ST77XX_BLACK);
  tft.setRotation(2);
  
  pinMode(w, INPUT_PULLUP);
  pinMode(s, INPUT_PULLUP);             //Botoes como pullup
  pinMode(d, INPUT_PULLUP);
  pinMode(a, INPUT_PULLUP);
  randomSeed(analogRead(A0));
  aleatorio= random(4);
  loading_screem();

EEPROM.get(0,jogador);
}

/////////////////////////////////////////////////VOID LOOP////////////////////////////////////////////////////////////
void loop() {
uni_char = 0;
cursor(75,90);

tela_inicio();

if ( digitalRead(d) == LOW ) {                        //SE O BOTAO 'D' FOR LOW ENTRA NA CONDICAO

    tft.fillScreen(ST77XX_BLACK);                       //VERIFICANDO O NUMERO DE 'Y' TROCA CASE PARA CASE DEPOIS DE ENTRAR NO 'IF'
  
    switch (cursor_y) {

      case 0:            //INICIAR
Vtempo = millis();
        do {

  tft.fillRect(70,0,120,15,ST77XX_BLACK);
  tft.setCursor(65,2);
  tft.setTextColor(ST77XX_WHITE);
  tft.setTextSize(1);
  tft.print("Pontos:");
  tft.print(pontos);
  tft.print("  Vidas:");
  tft.print(vidas);
          jogo();


        } while (vidas>0);      
        tft.fillScreen(0);
        do{
        jo_salvamento();
        }while(uni_char<3);
        pontos = 0;
        break;
///////////////////////////////////////
///////////////////////////////////////
      case 30:           //RANKING

        do {

          rank();

        } while (digitalRead(a) == HIGH);
        tft.fillScreen(0);
        break;
///////////////////////////////////////
///////////////////////////////////////
      case 60:           //Credditos

        do {

          creditos();

        } while (digitalRead(a) == HIGH);
        add_x = 240;
        tft.fillScreen(0);
        break;
///////////////////////////////////////
///////////////////////////////////////
      case 90:          //SAIR
      tft.fillScreen(0);
            tft.setCursor(10, 100);
    tft.setTextSize(3); 
    tft.print("###SAINDO###");
    delay(3000);
        do {

          sair();
       

        } while (digitalRead(a) == HIGH);
        delay(50);
    }
  }
}

void loading_screem() {


   tft.drawBitmap(0,0,pilar_cima,255,17,ST77XX_WHITE);
  tft.drawBitmap(230,0,pilar_dir,12,255,ST77XX_WHITE);
  tft.drawBitmap(0,0,pilar_esq,9,255,ST77XX_WHITE);
  tft.drawBitmap(0,225,pilar_baixo,255,15,ST77XX_WHITE);
  //tft.drawRect(4, 40, 240, 240,ST77XX_WHITE);

  tft.setCursor(20, 40);
  tft.setTextColor(ST77XX_ORANGE);
  tft.setTextSize(4);
  tft.print("Camaroes ");

  tft.setCursor(20, 150);
  tft.setTextColor(ST77XX_ORANGE);
  tft.setTextSize(4);
  tft.print("Acordados");
do{
  tft.fillRect(40, 100, loading_line, 25, ST77XX_RED);
  loading_line = loading_line+2;
}while(loading_line<180);
tft.fillScreen(ST77XX_BLACK);
}

/////////////////////////////////PRINTAR O MENU PRINCIPAL////////////////////////////////
void tela_inicio(void) {
  tft.drawBitmap(150,180,berglot,25,30,ST77XX_WHITE);
  tft.drawBitmap(0,0,pilar_cima,255,17,ST77XX_WHITE);
  tft.drawBitmap(230,0,pilar_dir,12,255,ST77XX_WHITE);
  tft.drawBitmap(0,0,pilar_esq,9,255,ST77XX_WHITE);
  tft.drawBitmap(0,225,pilar_baixo,255,15,ST77XX_WHITE);
  tft.drawBitmap(50,20,nome_jogo,144,43,ST77XX_WHITE);

  tft.setCursor(90, 90);
  tft.setTextColor(ST77XX_WHITE);
  tft.setTextSize(2);
  tft.print("Iniciar");

  tft.setCursor(90, 120);
  tft.setTextColor(ST77XX_WHITE);
  tft.setTextSize(2);
  tft.print("Ranking");

  tft.setCursor(90, 150);
  tft.setTextColor(ST77XX_WHITE);
  tft.setTextSize(2);
  tft.print("Creditos");

  tft.setCursor(90, 180);
  tft.setTextColor(ST77XX_WHITE);
  tft.setTextSize(2);
  tft.print("Sair");

  tft.setCursor(190, 230);
  tft.setTextColor(ST77XX_WHITE);
  tft.setTextSize(1);
  tft.print(versao);

  tft.setCursor(70, 90 + cursor_y); //ALTERANDO A VARIAVEL 'CURSOR_Y' VOCE MUDA AS cursorNADAS
  tft.setTextColor(ST77XX_WHITE);
  tft.setTextSize(2);
  tft.print(">");

}




///////////////////////////////////////////cursor///////////////////////////////////////////////////////////////
void cursor(int cursor_x,int cursor_my) {
  if (  digitalRead(w) == LOW ) {                     //DIMINUI O VALOR DE Y  FAZENDO COM QUE SE ELEVE ↑↑↑
    if ( cursor_y <= cursor_my && cursor_y >= 30 ) {            //CONDICAO PARA QUE O BLOCO NAO SAIA DO LIMITE DE BAIXO
      tft.fillRect(cursor_x-5, 90 + cursor_y, 18, 18, ST77XX_BLACK);
      cursor_y -= 30;
      delay(100);
    } else {
      tft.fillRect(cursor_x-5, 90 + cursor_y, 18, 18, ST77XX_BLACK);
      cursor_y = cursor_my;
      delay(100);
    }
  }
  if ( digitalRead(s) == LOW ) { //AUMENTA O VALOR DE +Y  FAZENDO COM QUE CAIA ↓↓↓
    if ( cursor_y >= 0 && cursor_y <= cursor_my-30 ) {    //CONDICAO PARA QUE O BLOCO NAO SAIA DO LIMITE DE SIMA
      tft.fillRect(cursor_x-5, 90 + cursor_y, 18, 18, ST77XX_BLACK);
      cursor_y += 30;
      delay(100);
    } else {
      tft.fillRect(cursor_x-5, 90 + cursor_y, 18, 18, ST77XX_BLACK);
      cursor_y = 0;
      delay(100);
    }
  }
}

// /// // /// // /// // /// // MOSTRAR RANK // /// // /// // /// // /// //

void rank() {
  int add_y = 0;
  tft.drawLine(1, 25, 240, 25, ST77XX_WHITE);
  tft.drawBitmap(0,0,pilar_cima,255,17,ST77XX_WHITE);
  tft.drawBitmap(230,0,pilar_dir,12,255,ST77XX_WHITE);
  tft.drawBitmap(0,0,pilar_esq,9,255,ST77XX_WHITE);
  tft.drawBitmap(0,225,pilar_baixo,255,15,ST77XX_WHITE);

  tft.setCursor(10, 10);
  tft.setTextColor(ST77XX_WHITE);
  tft.setTextSize(2);
  tft.print("SUA PONTUACAO");


  for(int i=0;i<3;i++){
    for (int f=0;f<3;f++){
      if(jogador[i].pontos>jogador[f].pontos){
         aux[0]=jogador[i];
        jogador[i] = jogador[f];
        jogador[f] = aux[0];
      }
    }
  }

  for (int i = 0 ; i < 3 ; i++) { //LOOP PARA PRINTAR NOMES E PONTOS
    tft.setCursor(15, 90 + add_y);
    tft.setTextColor(ST77XX_WHITE);
    tft.setTextSize(2);
    tft.print(jogador[i].nome);

    tft.setCursor(150, 90 + add_y);
    tft.setTextColor(ST77XX_WHITE);
    tft.setTextSize(2);
    tft.print(jogador[i].pontos);

    add_y += 30;
  }
}
// /// // /// // /// // /// // CREDITOS  // /// // /// // /// // /// //

void creditos() {

   tft.drawBitmap(0,0,pilar_cima,255,17,ST77XX_WHITE);
  tft.drawBitmap(230,0,pilar_dir,12,255,ST77XX_WHITE);
  tft.drawBitmap(0,0,pilar_esq,9,255,ST77XX_WHITE);
  tft.drawBitmap(0,225,pilar_baixo,255,15,ST77XX_WHITE);
  
  tft.setCursor(40, 15);
  tft.setTextColor(ST77XX_WHITE);
  tft.setTextSize(3);
  tft.print("Creditos");

  tft.setCursor(add_x+2, 105);
  tft.setTextColor(ST77XX_BLACK);
  tft.setTextSize(2);
  tft.print("Nicolas");
 tft.setCursor(add_x, 105);
  tft.setTextColor(ST77XX_WHITE);
  tft.setTextSize(2);
  tft.print("Nicolas");

  tft.setCursor(add_x+2, 90);
  tft.setTextColor(ST77XX_BLACK);
  tft.setTextSize(2);
  tft.print("Srum Master");
  tft.setCursor(add_x, 90);
  tft.setTextColor(ST77XX_SLPIN);
  tft.setTextSize(2);
  tft.print("Srum Master");

  tft.setCursor(add_x+2, 135);
  tft.setTextColor(ST77XX_BLACK);
  tft.setTextSize(2);
  tft.print("Gustavo");
   tft.setCursor(add_x, 135);
  tft.setTextColor(ST77XX_WHITE);
  tft.setTextSize(2);
  tft.print("Gustavo");

  tft.setCursor(add_x+2, 120);
  tft.setTextColor(ST77XX_BLACK);
  tft.setTextSize(2);
  tft.print("Programador");
   tft.setCursor(add_x, 120);
  tft.setTextColor(ST77XX_GREEN);
  tft.setTextSize(2);
  tft.print("Programador");


  tft.setCursor(add_x+2, 165);
  tft.setTextColor(ST77XX_BLACK);
  tft.setTextSize(2);
  tft.print("Pedro");
    tft.setCursor(add_x, 165);
  tft.setTextColor(ST77XX_WHITE);
  tft.setTextSize(2);
  tft.print("Pedro");

  tft.setCursor(add_x+2, 150);
  tft.setTextColor(ST77XX_BLACK);
  tft.setTextSize(2);
  tft.print("Programador");
    tft.setCursor(add_x, 150);
  tft.setTextColor(ST77XX_GREEN);
  tft.setTextSize(2);
  tft.print("Programador");

  tft.setCursor(add_x+2, 195);
  tft.setTextColor(ST77XX_BLACK);
  tft.setTextSize(2);
  tft.print("Guilherme");
    tft.setCursor(add_x, 195);
  tft.setTextColor(ST77XX_WHITE);
  tft.setTextSize(2);
  tft.print("Guilherme");

  tft.setCursor(add_x+2, 180);
  tft.setTextColor(ST77XX_BLACK);
  tft.setTextSize(2);
  tft.print("Desing");
    tft.setCursor(add_x, 180);
  tft.setTextColor(ST77XX_RED);
  tft.setTextSize(2);
  tft.print("Desing");

  
    add_x =add_x-2;
    
  

}


void sair() {

tft.fillScreen(0);
EEPROM.put(0,jogador);
}

void inimigos(){
  while(millis()-tempo1>1000){                      //velocidade de spawn dos inimigos
    
  switch(aleatorio){
    case 0:
    tiro1 = 1;
    tft.drawBitmap(190,30,wizardanim,23,30,ST77XX_BLUE);
    
  break;
    case 1:
    tiro2 = 1;
   tft.drawBitmap(190,80,wizardanim,23,30,ST77XX_BLUE);
break;
    case 2:
    tiro3 = 1;
    tft.drawBitmap(190,130,wizardanim,23,30,ST77XX_BLUE);
break;
    case 3:
    tiro4 = 1;
    tft.drawBitmap(190,180,wizardanim,23,30,ST77XX_BLUE);
break;
  }
  tempo1 = millis();
  }
}

void flecha_inimigoI(int inimigoY){

    
    tft.setCursor(160 - flecha_I, inimigoY );
    tft.setTextColor(ST77XX_WHITE);
    tft.setTextSize(1); 
    tft.print("<--");
    tft.setCursor((160+velocidade) - flecha_I,inimigoY );
    tft.setTextColor(ST77XX_BLACK);
    tft.setTextSize(1); 
    tft.print("<--");    
    flecha_I = flecha_I+velocidade;
   
}

void flecha_inimigoII(int inimigoY){

    
    tft.setCursor(160 - flecha_II, inimigoY );
    tft.setTextColor(ST77XX_WHITE);
    tft.setTextSize(1); 
    tft.print("<--");
    tft.setCursor((160+velocidade) - flecha_II,inimigoY );
    tft.setTextColor(ST77XX_BLACK);
    tft.setTextSize(1); 
    tft.print("<--");
    flecha_II= flecha_II +velocidade;
    
}

void flecha_inimigoIII(int inimigoY){

    
    tft.setCursor(160 - flecha_III, inimigoY );
    tft.setTextColor(ST77XX_WHITE);
    tft.setTextSize(1); 
    tft.print("<--");
    tft.setCursor((160+velocidade)- flecha_III,inimigoY );
    tft.setTextColor(ST77XX_BLACK);
    tft.setTextSize(1); 
    tft.print("<--");
    flecha_III = flecha_III +velocidade;
    }


void flecha_inimigoIV(int inimigoY){

    
    tft.setCursor(160 - flecha_IV, inimigoY );
    tft.setTextColor(ST77XX_WHITE);
    tft.setTextSize(1); 
    tft.print("<--");
    tft.setCursor((160+velocidade) - flecha_IV,inimigoY );
    tft.setTextColor(ST77XX_BLACK);
    tft.setTextSize(1); 
    tft.print("<--");
    flecha_IV= flecha_IV+velocidade;
    
}


void hitbox_inimigos(int inimigoY){
  if(hitbox_inimigo>130 && cursor_yJogo == inimigoY){
    tft.drawBitmap(190,inimigoY+30,wizardanim,23,30,ST77XX_BLACK);
    hitbox_inimigo = 0;
    if(inimigo==true && cursor_yJogo == inimigoY){
      pontos++;
      inimigo = false;
    }
    
    }
  }


void hitbox_personagem(){
if(cursor_yJogo==0&&flecha_I>=130&&flecha_I<131){
  vidas = vidas-1;
  }
  if(pontos>2000){
    if (cursor_yJogo==0&&flecha_I>=130&&flecha_I<134){
      vidas = vidas-1;}
  }
  


if(cursor_yJogo==50&&flecha_II>=130&&flecha_II<131){  
  vidas = vidas-1;
}
  if(pontos>2000){
    if (cursor_yJogo==50&&flecha_II>=130&&flecha_II<134){
      vidas = vidas-1;}
    }
    
if(cursor_yJogo==100&&flecha_III>=130&&flecha_III<131){ 
  vidas = vidas-1;
}
  if(pontos>2000){
    if (cursor_yJogo==100&&flecha_III>=130&&flecha_III<134){
      vidas = vidas-1;}
    }
if(cursor_yJogo==150&&flecha_IV>=130&&flecha_IV<131){
  
  vidas = vidas-1;
  
}
if(pontos>2000){
    if (cursor_yJogo==150&&flecha_IV>=130&&flecha_IV<134){
      vidas = vidas-1;}
    }
}



void flecha_Personagem(){

if (digitalRead(d)==0){
  do{
    tft.setCursor(30 + flecha_P,cursor_yJogo+30);
    tft.setTextColor(ST77XX_WHITE);
    tft.setTextSize(1); 
    tft.print("-->");
    tft.setCursor(25 + flecha_P,cursor_yJogo+30 );
    tft.setTextColor(ST77XX_BLACK);
    tft.setTextSize(1); 
    tft.print("-->");
    flecha_P = flecha_P + 5;
    hitbox_inimigo = flecha_P;
  }while(flecha_P < 205);
  tft.setCursor(0,cursor_yJogo + 38);
  tft.setTextColor(ST77XX_BLACK);
  tft.setTextSize(1); 
  tft.print("->"); 

  
}

flecha_P = 0;

}


void jogo(){

for (int i=60;i<211;i++){
  tft.drawLine(0,i,240,i,ST77XX_WHITE);                           //desenhando o chao
  i=i+49;
 }

tft.drawBitmap(15,30+cursor_yJogo,berglot,25,30,ST77XX_RED);     //desenha o personagem
flecha_Personagem();                                              //ataque do personagem


inimigos();                                                     //desenha os inimigos & lanca projetils
dificuldade();


hitbox_inimigos(0);
hitbox_inimigos(50);                                            //verifica se os inimigos forama atingidos
hitbox_inimigos(100);
hitbox_inimigos(150);




if (digitalRead(s)==0 && millis()-tempo3>500){
  if (0 <= cursor_yJogo && cursor_yJogo <= 100){                          //movimentacao do personagem para baixo
    tft.drawBitmap(15,30 + cursor_yJogo,berglot,25,30,ST77XX_BLACK);
    cursor_yJogo = cursor_yJogo + 50;
    
    }tempo3 = millis();
}

if (digitalRead(w)==0&& millis()-tempo3>500){
  if (150 >= cursor_yJogo && cursor_yJogo >= 50){                          //movimentacao do personagem para cima
    tft.drawBitmap(15,30 + cursor_yJogo,berglot,25,30,ST77XX_BLACK);
    cursor_yJogo = cursor_yJogo - 50;
   
  }tempo3 = millis();

}

  if(tiro1 == 1){
  flecha_inimigoI(30);
  if(flecha_I>180){
    flecha_I = 0;
    tiro1 = 0;}
  }
  
  if(tiro2 == 1){
  flecha_inimigoII(80);
  if(flecha_II>180){
    flecha_II = 0;
    tiro2 = 0;}
  }
  
if(tiro3 == 1){
  flecha_inimigoIII(130);
  if(flecha_III>180){
    flecha_III = 0;
    tiro3 = 0;}
}
if(tiro4 == 1){
  flecha_inimigoIV(180);
  if(flecha_IV>180){
    flecha_IV = 0;
    tiro4 = 0;} 
 }




aleatorio= random(4);


pontos++;
hitbox_personagem();



}







void dificuldade(){
  if(pontos>500){
    velocidade = 3;
  }
  if (pontos>1000 ){
    velocidade = 5;
  }
   if (pontos>2000 ){
    velocidade = 7;
  }
   if (pontos>2500 ){
    velocidade = 10;
  }
  if (pontos>3000 ){
    velocidade = 11;
  }
  if (pontos>3500 ){
    velocidade = 13
    ;
  }
}

///////////////////////////////////////SELECIONAR JOGADOR///////////////////////////////////////
void jo_salvamento(void) {
  int add_y = 0;
  short k=0;
  vidas = 4;
  flecha_I=0;flecha_II=0;flecha_III=0;flecha_IV=0;
  tiro1 = 0;tiro2 = 0;tiro3 = 0;tiro4 = 0;

    tft.drawBitmap(0,0,pilar_cima,255,17,ST77XX_WHITE);
  tft.drawBitmap(230,0,pilar_dir,12,255,ST77XX_WHITE);
  tft.drawBitmap(0,0,pilar_esq,9,255,ST77XX_WHITE);
  tft.drawBitmap(0,225,pilar_baixo,255,15,ST77XX_WHITE);
  tft.drawLine(1, 30, 240, 30, ST77XX_WHITE);
  tft.drawLine(1, 32, 240, 32, ST77XX_WHITE);

  tft.setCursor(45, 13);
  tft.setTextColor(ST77XX_WHITE);
  tft.setTextSize(2);
  tft.print("Fim de Jogo");


  if (digitalRead(d) == LOW) delay(100); //CONTROLE PARA NAO PULAR A TELA
  
  tft.drawRect(115, 97, 25, 36, ST77XX_WHITE);

  tft.setCursor(64, 180);
  tft.setTextColor(ST77XX_WHITE);
  tft.setTextSize(3);
  tft.print(ASCII_NUM);               // mostra  o valor em ASCII

  abc = ASCII_NUM;       // SOMA UM 'INT' EM UM 'CHAR' E ARMAZENA EM 'CHAR'
  
  tft.setCursor(120+k, 100);
  tft.setTextColor(ST77XX_WHITE);
  tft.setTextSize(3);
  tft.print(abc);


tft.setCursor(40,50+k);
  tft.setTextColor(ST77XX_WHITE);
  tft.setTextSize(3);
  tft.print(jogador[2].nome);
 
if (digitalRead(w) == LOW ) { //VERIFICA SE O PUSHBUTTON ESTA LIGADO
    if (ASCII_NUM < 90 ) {      //SE O VALOR DE ASCII ULTRAPASAR 90(Z)
      ASCII_NUM += 1;        //INCREMENTA 1 AO INT DE ASCII_NUM
      tft.fillScreen(ST77XX_BLACK);
      delay(150);
    } else {
      ASCII_NUM = 64; //RESETA A 65(A)
    }
  }
  if (digitalRead(s) == LOW ) {
    if (ASCII_NUM > 65) {      //SE O VALOR DE ASCII FOR MENOR QUE 64(A)
      ASCII_NUM -= 1;      //DECREMENTA 1 AO INT DE ASCII_NUM
      tft.fillScreen(ST77XX_BLACK);
      delay(150);
    } else {
      ASCII_NUM = 90; //RESETA A 90(Z)
    }
  }
   if(strcmp(jogador[2].nome,"Novo Jogo")==0){
   for(int i=0 ; i<9 ; i++){
    jogador[2].nome[i]='\0';
     
   }
  }


  if (digitalRead(d) == LOW) {
    tft.fillScreen(ST77XX_BLACK);
    if (uni_char <= 3) {
      
      jogador[2].nome[3] = {"\0"};
      jogador[2].nome[uni_char] = abc;
      uni_char += 1;
      delay(150);
    } 
  
  }

jogador[2].pontos = pontos;

  }
