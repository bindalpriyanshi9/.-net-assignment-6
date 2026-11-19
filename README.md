using System;
using System.Data;
using System.Data.SqlClient;
using System.Configuration;

namespace Q1_ReadStudents_Reader
{
    internal class Program
    {
        // Connection string is read from App.config (name = StudentDB)
        static readonly string ConnStr = ConfigurationManager.ConnectionStrings["StudentDB"].ConnectionString;

        static string ReadLineOrThrow()
        {
            string s = Console.ReadLine();
            if (s == null) throw new InvalidOperationException("No input available.");
            return s;
        }

        static int ReadInt(string prompt)
        {
            while (true)
            {
                Console.Write(prompt);
                int v;
                if (int.TryParse(ReadLineOrThrow(), out v)) return v;
                Console.WriteLine("  Invalid number, please try again.");
            }
        }

        static string ReadText(string prompt)
        {
            Console.Write(prompt);
            return ReadLineOrThrow();
        }

        static void PrintRow(object id, object name, object course, object age, object email)
        {
            Console.WriteLine("{0,-6} {1,-20} {2,-10} {3,-5} {4}", id, name, course, age, email);
        }

        static void PrintHeader()
        {
            PrintRow("ID", "Name", "Course", "Age", "Email");
            Console.WriteLine(new string('-', 62));
        }

        static void Main(string[] args)
        {
            try
            {
                using (SqlConnection con = new SqlConnection(ConnStr))
                {
                    con.Open();
                    Console.WriteLine("Connected to server '" + con.DataSource + "', database '" + con.Database + "'");
                    Console.WriteLine("Connection state: " + con.State);
                    Console.WriteLine();

                    SqlCommand cmd = new SqlCommand("SELECT StudentId, Name, Course, Age, Email FROM Student ORDER BY StudentId", con);

                    using (SqlDataReader dr = cmd.ExecuteReader())
                    {
                        PrintHeader();
                        int count = 0;
                        while (dr.Read())
                        {
                            PrintRow(dr["StudentId"], dr["Name"], dr["Course"], dr["Age"], dr["Email"]);
                            count++;
                        }
                        Console.WriteLine();
                        Console.WriteLine("Total records: " + count);
                    }
                }   // connection closed automatically by using
            }
            catch (SqlException ex)
            {
                Console.WriteLine("SQL error: " + ex.Message);
            }
            catch (Exception ex)
            {
                Console.WriteLine("Error: " + ex.Message);
            }

            Console.WriteLine("Press Enter to exit...");
            Console.ReadLine();
        }
    }
}



