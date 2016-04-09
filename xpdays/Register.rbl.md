<%
require 'net/smtp'
require 'cgi'

def input_field(label,name,length,value,required=false,fieldtype='text')
  val = '"' + CGI::escapeHTML(value) + '"'
  req = required ? "required='true'" : ""
  return "<tr><td>#{required ? '*' : ''}#{label}#{required ? '*' : ''}</td><td> <input type='#{fieldtype}' name='#{name}' size='#{length}' maxlength='#{length}' value=#{val} #{req}></td></tr>"
end

def autofocus_input_field(label,name,length,value,required=false,fieldtype='text')
  val = '"' + CGI::escapeHTML(value) + '"'
  req = required ? "required='true'" : ""
  return "<tr><td>#{required ? '*' : ''}#{label}#{required ? '*' : ''}</td><td> <input type='#{fieldtype}' name='#{name}' size='#{length}' maxlength='#{length}' value=#{val} #{req} autofocus></td></tr>"
end


def simple_input_field(label,name,length,value,required=false)
  val = '"' + CGI::escapeHTML(value) + '"'
  req = required ? "required='true'" : ""
  return "#{required ? '*' : ''}#{label}#{required ? '*' : ''} <input type='text' name='#{name}' size='#{length}' maxlength='#{length}' value=#{val} #{req}>"
end

def input_field_with_msg(label,name,length,value,msg,required=false)
  val = '"' + CGI::escapeHTML(value) + '"'
  req = required ? "required='true'" : ""
  return "<tr><td>#{required ? '*' : ''}#{label}#{required ? '*' : ''}</td><td> <input type='text' name='#{name}' size='#{length}' maxlength='#{length}' value=#{val} #{req}> #{msg}</td></tr>"
end

def input_fields(label,name,length,value,label2,name2,length2,value2,required=false)
  req = required ? "required='true'" : ""
  return "<tr><td>#{required ? '*' : ''}#{label}#{required ? '*' : ''}</td><td> <input type='text' name='#{name}' size='#{length}' maxlength='#{length} ' value='#{CGI::escapeHTML(value)}' #{req}> #{required ? '*' : ''}#{label2}#{required ? '*' : ''}&nbsp;<input type='text' name='#{name2}' size='#{length2}' maxlength='#{length2} ' value='#{CGI::escapeHTML(value2)}' #{req}></td></tr>"
end

def text_area(label,name,rows,columns,value)
  return "<tr><td colspan='2'>#{label}</td></tr><tr><td colspan='2'><textarea name='#{name}' rows='#{rows}' cols='#{columns}'>#{CGI::escapeHTML(value)}</textarea></td></tr>"
end

def radio(label,name,option,value)
  return "<input type='radio' name='#{name}' value='#{option}' #{value == option ? 'checked' : '' }>#{label}"
end

def checkbox(label,name,value)
  return "<tr><td colspan='2'><input type='checkbox' name='#{name}' #{value == 'on' ? 'checked' : ''}> #{label}</td></tr>"
end

def simple_checkbox(label,name,value)
  return "<input type='checkbox' name='#{name}' #{value == 'on' ? 'checked' : ''}> #{label}"
end

mail_sent = false
NB_PARTICIPANTS = 10
REGISTRATION_MAIL = "registrations@xpday.net"
#REGISTRATION_MAIL = "pvc@xpday.net"
REGISTRATION_DEFAULT = "Late"

name         = @request.parameter('name','').strip
email        = @request.parameter('email','').strip
company      = @request.parameter('company','').strip
vat          = @request.parameter('vat','').strip
reference    = @request.parameter('reference','').strip
address      = @request.parameter('address','').strip
zip          = @request.parameter('zip','').strip
city         = @request.parameter('city','').strip
country      = @request.parameter('country','').strip
phone        = @request.parameter('phone','').strip
agree        = @request.parameter('agree','').strip
print_invoice = @request.parameter('print_invoice','').strip
comments     = @request.parameter('comments','').strip
marketing    = @request.parameter('marketing','').strip
mailing        = @request.parameter('mailing','').strip

iznorobot = @request.parameter('iznorobot','').strip

participant_registration = Array.new
participant_name = Array.new
participant_email = Array.new
participant_food = Array.new
participant_dinner = Array.new

participants = 0
speakers     = 0

for p in 0..NB_PARTICIPANTS do
  p_registration = @request.parameter("participant#{p}_registration",REGISTRATION_DEFAULT).strip
  p_name         = @request.parameter("participant#{p}_name",'').strip

  if p_name.length > 0 then
    if p_registration =~ /Speaker/ then
      speakers += 1
    else
      participants += 1
    end
  end
  participant_registration.push p_registration
  participant_name.push p_name
  participant_email.push @request.parameter("participant#{p}_email",'').strip
  participant_food.push @request.parameter("participant#{p}_food",'').strip
  participant_dinner.push @request.parameter("participant#{p}_dinner",'none').strip

end

error = ''
if @request.params.length > 0 then
  error += '<li>Please fill in your name</li>' if name.empty?
  error += '<li>Please fill in your address</li>' if address.empty?
  error += '<li>Please fill in your zip code</li>' if zip.empty?
  error += '<li>Please fill in your city</li>' if city.empty?
  error += '<li>Please fill in your country</li>' if country.empty?
  error += '<li>Please fill in your email address</li>' if email.empty?
  error += '<li>Please confirm that you agree with our conditions</li>' if agree.empty?
  error += '<li>Please fill in the name of the first participant</li>' if participant_name[0].empty?
  error += '<li>Please fill in the email address of the first participant</li>' if participant_email[0].empty?
end

if email.length > 0 && error.length == 0 then
  content  = ''

  for p in 0..NB_PARTICIPANTS do
    if participant_name[p].length > 0 then
      content += "Participant #{p+1}:\n"
      content += "===================\n"
      content += "Name         = #{participant_name[p]}\n"
      content += "Email        = #{participant_email[p]}\n"
      content += "Phone        = #{phone}\n"
      content += "Company      = #{company}\n"
      content += "Country      = #{country}\n"

      content += "\nOrders:\n"
      content += "Registration    = #{participant_registration[p]}\n"
      content += "Pre-conf dinner = #{participant_dinner[p]}\n"
      content += "Food preference = #{participant_food[p]}\n\n"
    end
  end

  content += "\nSend invoice to\n"
  content += "==========\n"
  content += "Name         = #{name}\n"
  content += "Company      = #{company}\n"
  content += "VAT          = #{vat}\n"
  content += "Reference    = #{reference}\n"
  content += "Email        = #{email}\n"
  content += "Phone        = #{phone}\n"
  content += "Address      = #{address}\n"
  content += "Zip          = #{zip}\n"
  content += "City         = #{city}\n"
  content += "Country      = #{country}\n"
  content += "Printed invoice = #{print_invoice}\n"
  content += "Comment      = #{comments}\n\n"
  content += "Mailing   = #{mailing}\n\n"

  content += "How did you know about the conference ?\n"
  content += marketing
  content += "\n\n"

  content += "Submitted on #{Time.now.strftime("%d/%m/%Y %H:%M")}"

  message = "from: '#{name}' <#{email}>\nto: XP Days registration <registrations@xpday.net>\nsubject: XP Days 2015 registration\nDate: #{Time.now.strftime("%a, %d %b %Y %H:%M:%S")} +0100\n\n" + content

  begin
    @config.log("Sending registration message #{message}")
    Net::SMTP::start('localhost',25) do |smtp|
      smtp.send_message(message,email,REGISTRATION_MAIL)
    end
    mail_sent = true
  rescue Net::SMTPSyntaxError => syntax
    error += 'Please check the format of the email address<br>'
    @config.log("Encountered SMTP syntax exception #{syntax.inspect}")
  rescue Timeout::Error => timeout
    error += 'Could not send your message. Please check the format of the email address<br>'
    @config.log("Encountered Timeout exception #{timeout.inspect}")
  rescue Exception => ex
    error += 'Could not send your message. Please verify your the contents of your form and try again.'
    @config.log("Encountered exception #{ex.inspect}")
  end
end
%>

<script type="text/javascript">
<!--
function checkscript() {
     document.wiki2goedit.iznorobot.value = document.wiki2goedit.iznorobotty.value ;
       return true ;
       }
 // -->
</script>

<% if error.length > 0 then %>
  <div class="alert alert-block alert-error">
    <button type="button" class="close" data-dismiss="alert">&times;</button>
    <ul>
    <%= error %>
    </ul>
  </div>
<% end %>

<h3>Pricing</h3>
<table class="table table-bordered">
<tr><th>*Registration type*</th><th>*1 Day*</th><th>*2 Days*</th><th>*Available until*</th></tr>
<tr><td>Presenter</td><td> Free </td><td> Free </td><td> 20  November 2015</td></tr>
<tr><td><strike>Super Early</strike></td><td>N/A</td><td><strike>320,00 &euro; excl </strike></td><td><strike>0 tickets left</strike></td></tr>
<tr><td><strike>Early<strike></td><td><strike>160,00 &euro; excl VAT</strike></td><td><strike>350,00 &euro; excl VAT</strike></td><td><strike>16 October 2015</strike></td></tr>
<tr><td>*Late*</td><td>*180,00 &euro; excl VAT *</td><td>*380,00 &euro; excl VAT *</td><td>*SOLD OUT*</td></tr>
</table>

<% if mail_sent then %>

  <div class="alert alert-block alert-success">
  <strong>Thank you for registering. We will put you on the waiting list and contact you when someone cancels and tickets becme available.

Contact us at mailto:registrations@xpday.net if you have any questions or remarks about registration or payment.
</strong>
</div>

<% else %>


  <form method="post" action="/scripts/view/Xpday2015/XPDays/Register.rbl">
    <fieldset>
      <legend>Send invoice to</legend>
      <table>
        <%= autofocus_input_field('Name','name',64,name,true) %>
        <%= input_field('Company','company',64,company) %>
        <%= input_field('VAT','vat',18,vat,false) %>
        <%= input_field('Purchase order','reference',32,reference,false) %>
        <%= input_field('Address','address',64,address,true) %>
        <%= input_field('Zip','zip',16,zip,true) %>
        <%= input_field('City','city',64,city,true) %>
        <%= input_field('Country','country',64,country,true) %>
        <%= input_field('Email','email',64,email,true,'email') %>
        <%= input_field('Phone','phone',32,phone,false,'tel') %>
      </table>
    </fieldset>
    <%  for p in 0..NB_PARTICIPANTS do %>
        <%  show_reg = (p == 0 || participant_name[p].length > 0 ? '' : 'none') %>
        <div id="participant<%= p %>" style="display:<%= show_reg %>;">
          <fieldset>
            <legend>Participant <%= p +1 %></legend>
            <table>
              <tr><td colspan="2">
<!--    
          <tr><td>Super-Early Participant 2 days:</td><td><%= radio('320,00&euro;',"participant#{p}_registration",REGISTRATION_DEFAULT,participant_registration[p]) %></td></tr>
-->
              <tr><td>Late Participant 2 days:</td><td><%= radio('380,00&euro;',"participant#{p}_registration","Late",participant_registration[p]) %></td></tr>
              <tr><td>Late Participant Thursday:</td><td><%= radio('180,00&euro;',"participant#{p}_registration","Late_Thursday",participant_registration[p]) %></td></tr>
              <tr><td>Late Participant Friday:</td><td><%= radio('180,00&euro;',"participant#{p}_registration","Late_Friday",participant_registration[p]) %></td></tr>
<!--
              <tr><td>Sponsor participant:</td><td><%= radio(' Free',"participant#{p}_registration",'Sponsor',participant_registration[p]) %></td></tr>
-->
              <%= input_field('Name',"participant#{p}_name",64,participant_name[p],p == 0) %>
              <%= input_field('Email',"participant#{p}_email",64,participant_email[p],p == 0,'email') %>
              <%= input_field('Food preferences',"participant#{p}_food",64,participant_food[p]) %>
              <% if p < NB_PARTICIPANTS %>
                <tr><td colspan="2"><span  style="float:right;">Add another participant: <img src="/html/more_button.jpg" onclick="$('#participant<%= p+1 %>').show();" /></span></td></tr>
              <% end %>
            </table>
          </fieldset>
        </div>
      <% end %>
      <table>
        <%= text_area('Remarks or questions?','comments',3,80,comments) %>
        <%= text_area('How did you hear about the conference?','marketing',3,80,marketing) %>
        <%= checkbox('Add me to your conference announcement mailing list','mailing',mailing) %>
        <%= checkbox('*I have read the conditions below and I agree with them*','agree',agree) %>
        <tr><td colspan="2">
        <tr><td colspan="2"><input type="submit" name="Submit" id="Submit" value="Submit" >&nbsp;<input type="reset" name="Reset" id="Reset" value="Reset">&nbsp;&nbsp;*Note:* fields in bold must be filled in.
        </td></tr>
      </table>
<input type="hidden" name="iznorobot" value="" />
<input type="hidden" name="iznorobotty" value="e6dfe0f0fae71759d3cf463435cd59e86f37c0dccd2963c60913a69a62cdce93" />
    </form>
  <% end %>


*Terms and conditions*
   * After registration, you will receive a confirmation of your registration and an invoice. Don't forget to mention your VAT number.
   * If your company requires a Purchase Order, let us know so that we can put its reference number on the invoice.
   * We will email PDF invoices by default and will mail hardcopies upon request.
   * Your registration is confirmed as soon as {payment@XPDays/Payment} has been received. *All registrations must be paid before the conference starts!*
   * If you haven't received a confirmation from us that your payment was received, bring a proof of payment to the conference. See the {payment@XPDays/Payment} page for a list of payment methods. We will notify you of the receipt of your payment and confirm your registration within 3-4 business days. If you don't get a confirmation, please contact mailto:registrations@xpday.net
   * The number of places is limited to 150, registration is on a *first paid, first served basis*.
   * The conference fee covers attendance to all conference sessions and includes drinks and lunch.
   * The two-day ticket  includes a dinner on the evening of the first day. You can buy a separate dinner ticket if you only come one day or if you want to bring a guest to dinner.
   * The organizers have the right to cancel the event. In this case all paid fees will be refunded.
   * You can transfer your paid registration to someone else by sending the name, email address and any food preferences of the replacement to us at mailto:registrations@xpday.net
   * Contact mailto:registrations@xpday.net to cancel a registration. We will  refund the amount paid if we can transfer the registration to another participant. As the conference often sells out, we will probably find someone if you cancel at least two weeks before the conference.

*PRIVACY STATEMENT*

All information entered in this form will be stored in the database of {Agile Systems vzw@http://www.agilesystems.org/}.

This information is only used to handle the administration of the XP Days Benelux conference and to keep you informed about this and future events. In no case will this information be transferred to a third party.

You have the right to consult and to correct the information we hold. Contact mailto:info@agilesystems.org for any further information or inquiries.
$ALIAS:Pascal.Van.Cauwenberghe$
$NAME:Register for XP Days 2015$
$LASTMODIFIED:1445550476$
$CREATED_ON:1439793865$
$AUTHOR:Pascal.Van.Cauwenberghe$
