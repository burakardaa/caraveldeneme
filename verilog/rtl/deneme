`timescale 1ns / 1ps
//////////////////////////////////////////////////////////////////////////////////
// Company: 
// Engineer: 
// 
// Create Date: 10/22/2022 08:31:43 PM
// Design Name: 
// Module Name: fsm_otopilot
// Project Name: 
// Target Devices: 
// Tool Versions: 
// Description: 
// 
// Dependencies: 
// 
// Revision:
// Revision 0.01 - File Created
// Additional Comments:
// 
//////////////////////////////////////////////////////////////////////////////////


module oto_pilot
(

`ifdef USE_POWER_PINS
inout vssd1, 	
inout vccd1,
`endif
input wire clock,
input wire reset,

//input [9:0] gnss_i,
//input [9:0] altimetre_i,
//input [6:0] hedef_yukseklik_i,
//input yukseklik_bilgisi_i,
//output motor_o,
//output yesil_led_o,
//output kirmizi_led_o,

input  [18:0] io_in,
output [2:0] io_out,
output wire [2:0]io_oeb
);

assign io_oeb = 3'b000; 

localparam S_YUKSEKLIK_BEKLE    = 2'b00;
localparam S_ACIL_DURUS         = 2'b01;
localparam S_UCUS               = 2'b10;


wire [5:0] gnss_i; 
wire [5:0] altimetre_i; 
wire [5:0] hedef_yukseklik_i;
wire yukseklik_bilgisi_i; 

assign gnss_i = io_in[5:0];
assign altimetre_i = io_in[11:6];
assign  hedef_yukseklik_i = io_in[17:12];
assign  yukseklik_bilgisi_i = io_in [18]; 




// combinational assignments
reg hedef_yukseklik_hata;
reg motor_ac;
reg [5:0] sensor_fark;
reg [7:0] sensor_ortalama;

// registers
reg [1:0] state;
reg [1:0] hata_sayaci;
reg [5:0] atanan_yukseklik;
reg yesil_led;
reg kirmizi_led;
reg motor;


always @(*) begin
    if (hedef_yukseklik_i < 10 || hedef_yukseklik_i > 55) begin
        hedef_yukseklik_hata = 1'b1;
    end
    else begin
        hedef_yukseklik_hata = 1'b0;
    end

    if (gnss_i > altimetre_i) begin
        sensor_fark = gnss_i - altimetre_i;    
    end
    else begin
        sensor_fark = altimetre_i - gnss_i;
    end

    if (sensor_fark > 9) begin
        sensor_ortalama     = gnss_i;
    end
    else begin
        sensor_ortalama     = (gnss_i + altimetre_i) >> 1;        
    end
    
    motor_ac = 0;
    if (state == S_UCUS) begin
        if (sensor_ortalama >= atanan_yukseklik) begin
            motor_ac = 1'b0;
        end
        else begin
            motor_ac = 1'b1;
        end
    end

end

always @(posedge clock) begin
    if (reset == 1'b1) begin
        state               <= S_YUKSEKLIK_BEKLE;
        hata_sayaci         <= 2'b00;
        atanan_yukseklik    <= 6'b000000;
        yesil_led           <= 1'b0;
        kirmizi_led         <= 1'b0;
        motor               <= 1'b0;
    end
    else begin
        case (state)
            S_YUKSEKLIK_BEKLE : begin
                if (yukseklik_bilgisi_i == 1'b1) begin
                    if (hedef_yukseklik_hata == 1'b0) begin
                        atanan_yukseklik    <= hedef_yukseklik_i;
                        state               <= S_UCUS;
                    end
                    else begin
                        if (hata_sayaci == 2'b10) begin
                            state       <= S_ACIL_DURUS;
                            kirmizi_led <= 1'b1;
                        end
                        else begin
                            hata_sayaci <= hata_sayaci + 2'b01;
                            state       <= S_YUKSEKLIK_BEKLE;
                        end
                    end
                end
                else begin
                    state <= S_YUKSEKLIK_BEKLE;
                end
            end
            S_ACIL_DURUS : begin
                //
		state <= S_YUKSEKLIK_BEKLE;
            end
            S_UCUS : begin
                if (motor_ac == 1'b1) begin
                   motor        <= 1'b1; 
                   yesil_led    <= 1'b0;
	        end
                else begin
                    motor       <= 1'b0;
                    yesil_led   <= 1'b1;
                end
            end

            default: begin
                state <= S_YUKSEKLIK_BEKLE;
            end
        endcase
    end
end

assign io_out[0]       = motor;
assign io_out[1]       = yesil_led;
assign io_out[2]      = kirmizi_led;

endmodule
