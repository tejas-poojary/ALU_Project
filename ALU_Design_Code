`define MUL
module ALU_rtl_design #(parameter n=8,m=4)(OPA,OPB,CIN,CLK,INP_VALID,RST,CMD,CE,MODE,COUT,OFLOW,RES,G,E,L,ERR);

input [n-1:0]OPA,OPB;
input CLK,RST,CIN,CE,MODE;
input [1:0]INP_VALID;
input [m-1:0]CMD;
output reg ERR,OFLOW,COUT;
`ifdef MUL
  output reg [2*n-1:0] RES;
`else
  output reg [n:0] RES;
`endif
output reg G,L,E;

localparam required_bits = $clog2(n);
reg [required_bits-1:0] shift_value;
reg [n-1:0] temp_a, temp_b;
reg temp_cin, temp_ce, temp_mode;
reg [m-1:0] temp_cmd;
reg [1:0] temp_invalid;
reg signed [n-1:0] signed_a, signed_b;
reg signed [n:0] temp_res;
reg [n-1:0] mult_a, mult_b;
//reg [m-1:0] mul_cmd;
reg [1:0] mul_delay;

localparam [m-1:0]ADD=4'd0,
                 SUB=4'd1,
                 ADD_CIN=4'd2,
                 SUB_CIN=4'd3,
                 INC_A=4'd4,
                 DEC_A=4'd5,
                 INC_B=4'd6,
                 DEC_B=4'd7,
                 CMP=4'd8,
                 INC_MUL=4'd9,
                 SHIFT_MUL=4'd10,
                 SIGN_ADD=4'd11,
                 SIGN_SUB=4'd12,
                 AND=4'd0,
                 NAND=4'd1,
                 OR=4'd2,
                 NOR=4'd3,
                 XOR=4'd4,
                 XNOR=4'd5,
                 NOT_A=4'd6,
                 NOT_B=4'd7,
                 SHR1_A=4'd8,
                 SHL1_A=4'd9,
                 SHR1_B=4'd10,
                 SHL1_B=4'd11,
                 ROL_A_B=4'd12,
                 ROR_A_B=4'd13;

always @(posedge CLK or posedge RST)
 begin
  if (RST)
   begin
    temp_a <= 0;
    temp_b <= 0;
    temp_cin <= 0;
    temp_ce <= 0;
    temp_mode <= 0;
    temp_cmd <= 0;
    temp_invalid <= 0;
    mult_a <= 0;
    mult_b <= 0;
    mul_delay <= 0;
   end
  else
   begin
    temp_a <= OPA;
    temp_b <= OPB;
    temp_cin <= CIN;
    temp_ce <= CE;
    temp_mode <= MODE;
    temp_cmd <= CMD;
    temp_invalid <= INP_VALID;
     if (CE && MODE && INP_VALID == 2'b11 && !mul_delay && (CMD == INC_MUL || CMD == SHIFT_MUL))
      begin
       mult_a <= OPA;
       mult_b <= OPB;
       mul_delay <= 2;
      end
     else if (mul_delay > 0)
      begin
       mul_delay <= mul_delay - 1;
      end
   end
end

always @(posedge CLK or posedge RST)
 begin
  if(RST)
    begin
     `ifdef MUL
        RES <= {2*n{1'b0}};
     `else
        RES <= {n+1{1'b0}};
     `endif
     OFLOW<=1'b0;
     ERR<=1'b0;
     COUT<=1'b0;
     G<=1'b0;
     L<=1'b0;
     E<=1'b0;
    end
  else
    begin
     if(temp_ce)
       begin
        if(temp_mode&& !mul_delay)
         begin
           `ifdef MUL
              RES <= {2*n{1'b0}};
           `else
              RES <= {n+1{1'b0}};
           `endif
           OFLOW<=1'b0;
           ERR<=1'b0;
           COUT<=1'b0;
           G<=1'b0;
           L<=1'b0;
           E<=1'b0;
            if(temp_invalid==2'b00)
              begin
                `ifdef MUL
                  RES <= {2*n{1'b0}};
                `else
                  RES <= {n+1{1'b0}};
                `endif
                OFLOW<=1'b0;
                ERR<=1'b0;
                COUT<=1'b0;
                G<=1'b0;
                L<=1'b0;
                E<=1'b0;
              end
            else if(temp_invalid==2'b01)
              begin
                case(temp_cmd)
                  INC_A:begin
                          if(temp_a=={n{1'b1}})
                            begin
                             RES<=temp_a+1;
                             COUT<=1'b1;
                            end
                          else
                             RES<=temp_a+1;
                        end
                  DEC_A:begin
                          if(temp_a=={n{1'b0}})
                            begin
                             RES<=temp_a-1;
                             COUT<=1'b1;
                            end
                          else
                            RES<=temp_a-1;
                        end
                  default:begin
                           `ifdef MUL
                             RES <= {2*n{1'b0}};
                           `else
                             RES <= {n+1{1'b0}};
                           `endif
                           OFLOW<=1'b0;
                           ERR<=1'b0;
                           COUT<=1'b0;
                           G<=1'b0;
                           L<=1'b0;
                           E<=1'b0;
                          end
                endcase
              end
            else if(temp_invalid==2'b10)
              begin
                case(temp_cmd)
                  INC_B:begin
                         RES<=temp_b+1;
                         COUT<=(temp_b=={n{1'b1}})?1:0;
                        end
                  DEC_B:begin
                         RES<=temp_b-1;
                         COUT<=(temp_b==1'b0)?1:0;
                        end
                  default:begin
                           `ifdef MUL
                              RES <= {2*n{1'b0}};
                           `else
                              RES <= {n+1{1'b0}};
                           `endif
                           OFLOW<=1'b0;
                           ERR<=1'b0;
                           COUT<=1'b0;
                           G<=1'b0;
                           L<=1'b0;
                           E<=1'b0;
                          end
                endcase
              end
            else
               begin
                 case(temp_cmd)
                   ADD:begin                     //ADD
                        RES<=temp_a+temp_b;
                        COUT<=RES[n]?1:0;
                       end
                   SUB:begin                     //SUB
                        RES<=temp_a-temp_b;
                        COUT<=(temp_a<temp_b)?1:0;
                       end
                   ADD_CIN:begin                     //ADD_CIN
                             RES<=temp_a+temp_b+temp_cin;
                             COUT<=RES[n]?1:0;
                           end
                   SUB_CIN:begin                     //SUB_CIN
                             RES<=temp_a-temp_b-temp_cin;
                             COUT<=((temp_a<temp_b)||((temp_a==temp_b)&&temp_cin))?1:0;
                           end
                   CMP:begin
                        `ifdef MUL
                           RES <= {2*n{1'b0}};
                        `else
                           RES <= {n+1{1'b0}};
                        `endif
                        E<=(temp_a==temp_b)? 1'b1:1'b0;
                        G<=(temp_a>temp_b)? 1'b1:1'b0;
                        L<=(temp_a<temp_b)? 1'b1:1'b0;
                       end
                   SIGN_ADD:begin
                              signed_a=$signed(temp_a);
                              signed_b=$signed(temp_b);
                              temp_res=signed_a+signed_b;
                              RES<=(temp_res);
                              OFLOW<=(signed_a[n-1] == signed_b[n-1]) && (temp_res[n-1] != signed_a[n-1]);
                              G<=(signed_a>signed_b)?1:0;
                              L<=(signed_a<signed_b)?1:0;
                              E<=(signed_a==signed_b)?1:0;
                            end
                   SIGN_SUB:begin
                              signed_a=$signed(temp_a);
                              signed_b=$signed(temp_b);
                              temp_res=signed_a-signed_b;
                              RES<=(temp_res);
                              OFLOW<= (signed_a[n-1] != signed_b[n-1]) && (temp_res[n-1] != signed_a[n-1]);
                              G<=(signed_a>signed_b)?1:0;
                              L<=(signed_a<signed_b)?1:0;
                              E<=(signed_a==signed_b)?1:0;
                            end
                   default:begin
                             `ifdef MUL
                               RES <= {2*n{1'b0}};
                             `else
                               RES <= {n+1{1'b0}};
                             `endif
                             OFLOW<=1'b0;
                             ERR<=1'b0;
                             COUT<=1'b0;
                             G<=1'b0;
                             L<=1'b0;
                             E<=1'b0;
                           end
                 endcase
               end
         end
        else            //else part to mode=0
          begin
            `ifdef MUL
               RES <= {2*n{1'b0}};
             `else
               RES <= {n+1{1'b0}};
             `endif
                OFLOW<=1'b0;
                ERR<=1'b0;
                COUT<=1'b0;
                G<=1'b0;
                L<=1'b0;
                E<=1'b0;
            if(temp_invalid==2'b00)
               begin
                `ifdef MUL
                  RES <= {2*n{1'b0}};
                `else
                  RES <= {n+1{1'b0}};
                `endif
                OFLOW<=1'b0;
                ERR<=1'b0;
                COUT<=1'b0;
                G<=1'b0;
                L<=1'b0;
                E<=1'b0;
               end
             else if(temp_invalid==2'b01)
               begin
                 case(temp_cmd)
                   NOT_A:RES<={1'b0,~temp_a};
                   SHR1_A:RES<={1'b0,temp_a>>1};
                   SHL1_A:RES<={1'b0,temp_a<<1};
                   default:begin
                          `ifdef MUL
                            RES <= {2*n{1'b0}};
                          `else
                            RES <= {n+1{1'b0}};
                          `endif
                           OFLOW<=1'b0;
                           ERR<=1'b0;
                           COUT<=1'b0;
                           G<=1'b0;
                           L<=1'b0;
                           E<=1'b0;
                          end
                 endcase
               end
             else if(temp_invalid==2'b10)
               begin
                 case(temp_cmd)
                   NOT_B:RES<={1'b0,~temp_b};
                   SHR1_B:RES<={1'b0,temp_b>>1};
                   SHL1_B:RES<={1'b0,temp_b<<1};
                   default:begin
                           `ifdef MUL
                             RES <= {2*n{1'b0}};
                           `else
                             RES <= {n+1{1'b0}};
                           `endif
                           OFLOW<=1'b0;
                           ERR<=1'b0;
                           COUT<=1'b0;
                           G<=1'b0;
                           L<=1'b0;
                           E<=1'b0;
                          end
                 endcase
               end
             else
               begin
                 case(temp_cmd)
                   AND:RES<={1'b0,(temp_a & temp_b)};
                   NAND:RES<={1'b0,~(temp_a & temp_b)};
                   OR:RES<={1'b0,(temp_a | temp_b)};
                   NOR:RES<={1'b0,~(temp_a | temp_b)};
                   XOR:RES<={1'b0,(temp_a ^ temp_b)};
                   XNOR:RES<={1'b0,~(temp_a ^ temp_b)};
                   ROL_A_B:begin
                            shift_value<=temp_b[required_bits-1:0];
                            RES<={1'b0,((temp_a<<shift_value)|(temp_a>>n-shift_value))};
                            if(temp_b>n-1)
                             ERR<=1;
                            else
                             ERR<=0;
                           end
                   ROR_A_B:begin
                            shift_value<=temp_b[required_bits-1:0];
                            RES<={1'b0,((temp_a>>shift_value)|(temp_a<<n-shift_value))};
                            if(temp_b>n-1)
                              ERR<=1;
                             else
                              ERR<=0;
                           end
                   default:begin
                            `ifdef MUL
                              RES <= {2*n{1'b0}};
                            `else
                              RES <= {n+1{1'b0}};
                            `endif
                            OFLOW<=1'b0;
                            ERR<=1'b0;
                            COUT<=1'b0;
                            G<=1'b0;
                            L<=1'b0;
                            E<=1'b0;
                           end
                 endcase
               end
          end          //end of else mode=0
       end
     else
       begin
         `ifdef MUL
            RES <= {2*n{1'b0}};
         `else
            RES <= {n+1{1'b0}};
         `endif
         OFLOW<=1'b0;
         ERR<=1'b0;
         COUT<=1'b0;
         G<=1'b0;
         L<=1'b0;
         E<=1'b0;
       end
  end
     if (mul_delay==1)
      begin
        `ifdef MUL
          if(INC_MUL)
            begin
               RES<=(mult_a+1)*(mult_b+1);
            end
          else
            begin
               RES<=(mult_a<< 1)*mult_b;
            end
        `endif
      end
   end
endmodule
