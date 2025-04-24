# newsletter
sending a newsletter with an unsubscribe link


import { serve } from "https://deno.land/std@0.168.0/http/server.ts";
import * as resend from "https://esm.sh/@resend/emails@0.4.0";
import { createClient } from "https://esm.sh/@supabase/supabase-js@2";

const corsHeaders = {
  "Access-Control-Allow-Origin": "*",
  "Access-Control-Allow-Headers": "authorization, x-client-info, apikey, content-type",
};

// Initialize Supabase client
const supabase = createClient(
  Deno.env.get("SUPABASE_URL") ?? "",
  Deno.env.get("SUPABASE_ANON_KEY") ?? ""
);

// Initialize Resend client
const resendClient = resend.Resend(Deno.env.get("RESEND_API_KEY"));

serve(async (req) => {
  if (req.method === "OPTIONS") {
    return new Response("ok", { headers: corsHeaders });
  }

  try {
    // Fetch subscriber emails from Supabase
    const { data: subscribers, error } = await supabase
      .from("users") // Replace with your table name
      .select("email");

    if (error) throw error;

    // Prepare email content
    const subject = "Your Monthly Newsletter";
    const html = `
      <h1>Hello from Your Brand!</h1>
      <p>Here's the latest news...</p>
      <p><a href="{{unsubscribe}}">Unsubscribe</a></p>
    `;

    // Send email to each subscriber
    for (const subscriber of subscribers) {
      await resendClient.emails.send({
        from: "newsletter@yourdomain.com",
        to: subscriber.email,
        subject,
        html,
        headers: {
          "List-Unsubscribe": `<mailto:unsubscribe@yourdomain.com?subject=unsubscribe>`,
          "List-Unsubscribe-Post": "List-Unsubscribe=One-Click",
        },
      });
    }

    return new Response(
      JSON.stringify({ message: "Emails sent successfully" }),
      { headers: { "Content-Type": "application/json", ...corsHeaders } }
    );
  } catch (error) {
    return new Response(
      JSON.stringify({ error: error.message }),
      { headers: { "Content-Type": "application/json", ...corsHeaders }, status: 400 }
    );
  }
});
