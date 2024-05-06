====================
Accumulator Register
====================

         .. rubric:: Introduction
            :name: introduction
            :class: rvps334

         The accumulator is a register in which intermediate arithmetic
         logic unit results are stored. While a device is working on
         multi-step operations, intermediate values are sent to the
         accumulator and then overwritten as needed. Accumulator adds a
         value with each write but reads back the total accumulated
         result of the sum of the current value and the previous value
         of the field.

         “accumulate=true” property has been used, so that the register
         can sum the previously written value and the current updated
         values of the fields flop.

         The register would be written from the software side to update
         the value. The HW access of fields would be NA or the RO for
         this feature.

         Note: This property is only supported at reg level.

         Example:      
          \ `IDS-NG <https://www.portal.agnisys.com/release/idsdocs/examples/properties/accumulator/accumulator.idsng.zip>`__\ 
              
          \ `IDS-Word <https://www.portal.agnisys.com/release/idsdocs/examples/properties/accumulator/accumulator.docx>`__\ 
              
          \ `IDS-Excel <https://www.portal.agnisys.com/release/idsdocs/examples/properties/accumulator/accumulator.xls>`__\ 
              
          \ `SystemRDL <https://www.portal.agnisys.com/release/idsdocs/examples/properties/accumulator/accumulator.rdl>`__

         IDS-NG Register View

         .. image:: lib/NewItem4962.png

         IDS-NG Spreadsheet View

         .. image:: lib/NewItem4963.png

         SystemRDL

         | property accumulate{type = boolean; component = reg;
           };addrmap block1{
         |    reg reg1{
         |          accumulate = true;
         |          field field1{
         |                hw = na;   //hw can be(Read-Only or NA)
         |                sw = rw;
         |          };field1 f1[31:0] = 0;
         | };   reg1 r1;
         | };

         Generated accumulate UVM callback in the package file 

         //   \*.regmem.sv file

         | / Function : build
         |     virtual function void build();
         |         //define default map and add reg/regfiles
         |         default_map= create_map("default_map", 'h0, 4,
           UVM_BIG_ENDIAN, 1);        //R1
         |         r1 = block1_r1::type_id::create("r1");
         |         r1.configure(this, null, "r1");
         |         r1.build();
         |         default_map.add_reg( r1, 'h0, "RW");        //
           Registering callback class instances with register
           fields        begin
         |             acum_cb_class  block1_r1_f1;
         |             block1_r1_f1 = new( "block1_r1_f1",r1.f1);
         |             uvm_reg_field_cb::add(r1.f1,block1_r1_f1);        end        lock_model();
         |     endfunction

         //   \*_pkg.regmem.sv file

         | package block1_block_regmem_pkg;
         |     import uvm_pkg::*;
         |     `include
           "uvm_macros.svh"    /*----------------------------------------------------------
         |     Class       : acum_CB
         |     Description : UVM Callback class for secded register
         |     ------------------------------------------------------------*/    class
           acum_cb_class extends uvm_reg_cbs;
         |         uvm_reg_field m_estatus;        function new(string
           name, uvm_reg_field estatus);
         |             super.new();
         |             m_estatus = estatus;
         |         endfunction        virtual task pre_read(uvm_reg_item
           rw);
         |             if(rw.status == UVM_IS_OK) begin
         |                 void'(m_estatus.predict(m_estatus.get_mirrored_value()
           + m_estatus.get()));
         |             end
         |         endtask    endclass    `include
           "\ `block1.regmem.sv <http://block1.regmem.sv/>`__\ "
         | endpackage

         Generated RTL Output

         module block1_ids(

             

             accum_error,

             

             //APB signals

             pclk,   // Bus clock

             presetn,   // Reset

             psel,   // Select    : It indicates that the target device
         is selected and a data transfer is required

             penable,   // Enable    : This signal indicates the second
         and subsequent cycles of an APB transfer

             pwrite,   // Direction : This signal indicates an APB write
         access when HIGH and an APB read access when LOW

             pprot,   // Protection type : This signal indicates the
         normal, privileged, or secure protection level of the
         transaction

             . . . . .

         . . . . .

         reg r1_f1_overflow; // FIELD : f1

             reg r1_f1_q; // FIELD : f1

             output accum_error;

         . . . . .

         . . . . .

             always @(posedge clk)  begin

             if (!reset_l)

                 begin

                     r1_f1_q <= 1'd0;

                 end

             else

                 begin

                 if (r1_wr_valid) //F1 : SW Write

                     begin

                         {r1_f1_overflow,r1_f1_q} <= r1_f1_q + (wr_data
         [0]  & reg_enb  [0] ) \| (r1_f1_q & (~reg_enb  [0] ));

                     end

                 end

             end //end always

             assign r1_rd_data  = r1_rd_valid ? {31'h0, r1_f1_q} :
         32'd0;

             assign r1_overflow = r1_f1_overflow;

             assign rd_data_vld = rd_stb;

             assign rd_data = r1_rd_data;

             assign request = 1'b1;

             assign rd_wait = 1'b1;

             assign accum_error = r1_overflow;

             assign error = 1'b0;

         endmodule

         Created with the Personal Edition of HelpNDoc: \ `Experience
         the Power and Ease of Use of a Help Authoring
         Tool <https://www.helpndoc.com>`__

      .. container::
         :name: topic_footer

         .. container::
            :name: topic_footer_content

            © 2007 - 2023 Agnisys® Inc. All Rights Reserved.
            https://www.agnisys.com/submit-feedback/

.. container:: mask

.. container:: modal fade
   :name: hndModal

   .. container:: modal-dialog

      .. container:: modal-content

         .. container:: modal-header

            ×
            .. rubric:: 
               :name: hndModalLabel
               :class: modal-title

         .. container:: modal-body

         .. container:: modal-footer

            Close

.. container::
   :name: hnd-splitter
